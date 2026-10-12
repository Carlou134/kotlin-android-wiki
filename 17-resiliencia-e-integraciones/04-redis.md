# Redis

## En una frase

Redis es una base de datos en memoria, de clave y valor, muy rápida y compartida entre todas las instancias de tu backend; se usa sobre todo como **caché** con vencimiento (TTL) y para **contadores**, pero nunca como la fuente de verdad de tus datos.

-----

## Antes de empezar

Conviene que ya sepas:

* Qué es un puerto de salida: [Arquitectura hexagonal](../12-arquitectura/02-arquitectura-hexagonal.md).
* El problema de los límites por instancia: [Tokens de servicio y rate limiting](../16-seguridad/06-tokens-de-servicio-y-rate-limiting.md).

Palabras nuevas (también están en el [Glosario](Glosario.md)):

* **Redis:** Base de datos en memoria de clave y valor.
* **Caché:** Copia temporal de un dato caro de obtener, guardada en un lugar rápido.
* **TTL (time to live):** Tiempo después del cual una clave se borra sola.
* **Cache-aside:** Patrón en el que la app busca primero en la caché y, si no está, carga de la fuente y la guarda.
* **Invalidación:** Borrar o actualizar una entrada de caché cuando el dato original cambia.
* **Lettuce:** Cliente de Redis para la JVM, con soporte para corrutinas.

-----

## Para qué sirve

| Uso | Ejemplo |
| --- | --- |
| Caché | Guardar 5 minutos el resultado de una consulta pesada o de una API lenta |
| Contadores compartidos | Rate limiting global entre varias instancias |
| Datos de vida corta | Códigos de verificación que vencen en 10 minutos |

¿Por qué no simplemente un `Map` en memoria? Porque con 3 instancias, cada una tiene **su propio** mapa: tres cachés distintas, tres contadores distintos. Redis es **uno** solo, compartido por todas.

## Los comandos básicos

```
SET pedido:p1 "{...json...}" EX 300   # guarda con TTL de 300 segundos
GET pedido:p1                          # devuelve el valor, o nil si no existe o venció
DEL pedido:p1                          # borra
INCR visitas:2026-10-12                # suma 1 de forma atómica y devuelve el nuevo valor
EXPIRE visitas:2026-10-12 86400        # pone un TTL a una clave existente
```

Todos los comandos son **atómicos**: `INCR` desde 10 instancias a la vez nunca pierde una suma.

## Con Lettuce y corrutinas

```kotlin
val cliente = RedisClient.create("redis://localhost:6379")
val conexion = cliente.connect()

@OptIn(ExperimentalLettuceCoroutinesApi::class)
val redis = conexion.coroutines()

suspend fun guardar(clave: String, valor: String, ttl: Duration) {
  redis.set(clave, valor, SetArgs().ex(ttl.inWholeSeconds))
}

suspend fun leer(clave: String): String? = redis.get(clave)
```

Igual que el pool de la base o el cliente HTTP: **una** conexión compartida, creada al arrancar y cerrada al apagar. Lettuce permite que muchas corrutinas usen la misma conexión a la vez.

## El patrón cache-aside

```kotlin
class EstadisticasConCache(
  private val origen: EstadisticasRepository,
  private val cache: Cache,
) : EstadisticasRepository {

  override suspend fun resumen(empresaId: EmpresaId): Resumen {
    val clave = "estadisticas:${empresaId.value}"

    cache.leer(clave)?.let { return Json.decodeFromString(it) }   // 1. ¿está en caché?

    val resumen = origen.resumen(empresaId)                        // 2. no: cargar de la fuente
    cache.guardar(clave, Json.encodeToString(resumen), 5.minutes)  // 3. guardar con TTL
    return resumen
  }
}
```

Fíjate en el diseño: `EstadisticasConCache` implementa el **mismo puerto** que el repositorio real y lo envuelve. El use case no sabe si hay caché o no. Esto es el patrón **decorador**, y se conecta en el módulo de Koin.

## Las claves llevan el tenant

```kotlin
"estadisticas:${empresaId.value}"   // ✅
"estadisticas"                      // ❌ la empresa B ve las estadísticas de la empresa A
```

Una clave de caché sin el tenant es la forma más rápida de mezclar datos entre empresas, y es muy difícil de detectar en los tests. Lo viste en [RBAC y multi-tenancy](../16-seguridad/05-rbac-y-multitenancy.md).

## Invalidación: el problema difícil

Si el dato cambia en la base, la caché queda **vieja** hasta que vence. Opciones:

| Estrategia | Cómo | Cuándo |
| --- | --- | --- |
| Solo TTL | Aceptar datos viejos por unos minutos | Estadísticas, catálogos, cosas que toleran retraso |
| Borrar al escribir | Al actualizar en la base, `DEL` de la clave | Cuando el dato viejo causa problemas |

Con "borrar al escribir", **siempre** pon también un TTL: si un `DEL` falla, el TTL limita cuánto dura el dato viejo.

La regla para decidir: **¿qué pasa si el usuario ve el dato de hace 5 minutos?** Si la respuesta es "nada grave", alcanza con un TTL.

## Redis no es la fuente de verdad

Redis guarda todo en memoria. Aunque puede persistir en disco, se usa asumiendo que **puede perder datos**.

* Lo que no puedes perder va en PostgreSQL. Redis solo tiene **copias**.
* Si Redis se cae, la app debe seguir funcionando, más lenta pero funcionando: ante un error de Redis, ve directo a la fuente.

```kotlin
val enCache = try {
  cache.leer(clave)
} catch (e: CancellationException) {
  throw e
} catch (e: Exception) {
  logger.warn(e) { "Redis no disponible, se usa la fuente" }
  null
}
```

## Rate limiting global

El contador por ventana fija en Redis:

```kotlin
suspend fun permitir(clienteId: String, limite: Int, ventana: Duration): Boolean {
  val clave = "ratelimit:$clienteId:${Clock.System.now().epochSeconds / ventana.inWholeSeconds}"
  val cuenta = redis.incr(clave) ?: 0
  if (cuenta == 1L) redis.expire(clave, ventana.inWholeSeconds)
  return cuenta <= limite
}
```

Todas las instancias cuentan en la **misma** clave, así que el límite es global.

## Locks distribuidos: con cuidado

Redis también se usa para locks (`SET clave valor NX EX 30`: "créala solo si no existe"). Funciona, pero tiene los mismos problemas que un lease (puede vencer mientras trabajas) y otros propios (si Redis pierde datos, pierde el lock). Si ya tienes PostgreSQL, el [lease en una tabla](../15-persistencia/06-bloqueos-y-concurrencia.md) suele ser más simple y más seguro.

-----

## Resumen en 5 líneas

1. Redis es memoria de clave y valor compartida entre instancias; sus comandos son atómicos y las claves pueden vencer con TTL.
2. Cache-aside: buscar en caché, si no está cargar de la fuente y guardar con TTL; se implementa como decorador del mismo puerto.
3. Las claves llevan siempre el tenant; la invalidación más simple es un TTL corto, y borrar al escribir siempre con TTL de respaldo.
4. Redis no es la fuente de verdad: si falla, la app sigue funcionando contra la base.
5. Sirve para contadores globales (rate limiting); para locks, un lease en PostgreSQL suele ser más seguro.

-----

## Siguiente lección

[5. Almacenamiento S3 y MinIO](05-almacenamiento-s3-y-minio.md): dónde van los archivos.
