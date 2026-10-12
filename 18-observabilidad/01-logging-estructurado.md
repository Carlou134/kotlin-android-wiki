# Logging estructurado

## En una frase

Un log estructurado no es una frase suelta sino un **objeto JSON** con campos (nivel, mensaje, hora, `pedidoId`, `callId`...), así una herramienta puede buscar, filtrar y contar entre millones de líneas; en la JVM se arma con SLF4J, Logback, kotlin-logging y un encoder JSON.

-----

## Antes de empezar

Conviene que ya sepas:

* Excepciones y `CancellationException`: [Excepciones](../07-null-safety-y-errores/02-excepciones.md) y [Concurrencia estructurada](../10-corrutinas/03-concurrencia-estructurada.md).
* Lambdas: [Funciones de orden superior](../09-funcional/01-funciones-de-orden-superior.md).

Palabras nuevas (también están en el [Glosario](Glosario.md)):

* **Log:** Registro de algo que pasó en la aplicación.
* **Nivel de log:** Qué tan importante es un log: `ERROR`, `WARN`, `INFO`, `DEBUG`, `TRACE`.
* **SLF4J:** Fachada de logging de la JVM: la API que usa el código, sin atarse a una implementación.
* **Logback:** Implementación de SLF4J que decide qué se escribe, dónde y en qué formato.
* **Appender:** El destino de los logs en Logback: la consola, un archivo.
* **Encoder:** El formato en que un appender escribe cada log: texto o JSON.
* **Dato personal (PII):** Información que identifica a una persona: email, nombre, documento, teléfono.

-----

## El problema: logs que no se pueden buscar

```
2026-10-12 14:30:01 INFO  Pedido p-9 creado para cliente c-7 por 150.00
2026-10-12 14:30:02 ERROR Falló la sincronización del proveedor a: timeout
```

Se lee bien de a una línea. Pero con 3 instancias y millones de líneas por día, preguntas como "¿cuántas sincronizaciones fallaron por proveedor en la última hora?" o "todos los logs del cliente c-7" obligan a escribir expresiones regulares frágiles sobre texto libre.

El mismo log, **estructurado**:

```json
{"@timestamp":"2026-10-12T14:30:01.123Z","level":"ERROR","logger":"SincronizarProveedores",
 "message":"Falló la sincronización","proveedor":"a","error":"timeout","callId":"8f3c2a10"}
```

Ahora la herramienta de logs (Loki, Elasticsearch, CloudWatch, Datadog...) filtra por `level = ERROR AND proveedor = a` en milisegundos.

## Las piezas en la JVM

```
Tu código ──> kotlin-logging ──> SLF4J (API) ──> Logback (implementación) ──> encoder JSON ──> consola
```

* **SLF4J** es la interfaz. Las librerías (Ktor, Exposed, HikariCP) loguean a través de ella.
* **Logback** decide qué niveles se muestran, dónde y en qué formato.
* **kotlin-logging** es una capa fina sobre SLF4J con una API más idiomática para Kotlin.
* **logstash-logback-encoder** hace que Logback escriba JSON.

## Loguear con kotlin-logging

```kotlin
private val logger = KotlinLogging.logger {}

class SincronizarProveedores(...) {
  suspend fun ejecutar() {
    logger.info { "Sincronización iniciada" }
    // ...
    logger.debug { "Respuesta del proveedor: ${dto.resumen()}" }
  }
}
```

* `KotlinLogging.logger {}` toma el nombre de la clase o del archivo automáticamente.
* El mensaje va en una **lambda**: solo se evalúa si el nivel está activo. Con `DEBUG` apagado en producción, `dto.resumen()` ni se ejecuta.

## Niveles: cuál usar

| Nivel | Cuándo | Ejemplo |
| --- | --- | --- |
| `ERROR` | Algo falló y requiere atención de alguien | Error inesperado en una ruta, un job que no pudo ejecutar |
| `WARN` | Algo raro pero manejado; si se repite, investigar | Un proveedor respondió `503` y se reintentará |
| `INFO` | Hitos del negocio o del sistema | Servidor iniciado, sincronización terminada con 120 registros |
| `DEBUG` | Detalle para depurar, apagado en producción | Parámetros de una consulta, respuestas resumidas |
| `TRACE` | Detalle extremo | Casi nunca en código de aplicación |

Si **todo** es `ERROR`, nadie mira los errores. Un `ERROR` debería significar "alguien tiene que hacer algo".

## Logs en JSON con Logback

`src/main/resources/logback.xml`:

```xml
<configuration>
  <appender name="JSON" class="ch.qos.logback.core.ConsoleAppender">
    <encoder class="net.logstash.logback.encoder.LogstashEncoder"/>
  </appender>

  <logger name="com.zaxxer.hikari" level="WARN"/>
  <logger name="io.netty" level="WARN"/>

  <root level="INFO">
    <appender-ref ref="JSON"/>
  </root>
</configuration>
```

* Los logs van a la **consola** (stdout). En contenedores, la plataforma los recoge de ahí; la app no maneja archivos de log.
* Se baja el nivel de librerías ruidosas.
* En tu máquina, el JSON es incómodo de leer: es común tener un `logback.xml` con formato de texto para desarrollo y otro con JSON para los entornos desplegados, elegido con la propiedad `-Dlogback.configurationFile=...` al arrancar.

## Campos estructurados

El mensaje describe **qué** pasó; los datos van en **campos** separados:

```kotlin
import net.logstash.logback.argument.StructuredArguments.kv

logger.info("Sincronización terminada {} {} {}",
  kv("proveedor", proveedor.nombre),
  kv("registros", resultado.cantidad),
  kv("duracionMs", duracion.inWholeMilliseconds),
)
```

```json
{"message":"Sincronización terminada proveedor=a registros=120 duracionMs=840",
 "proveedor":"a","registros":120,"duracionMs":840, ...}
```

kotlin-logging también tiene una API fluida para agregar campos (`logger.atInfo { message = ...; payload = mapOf(...) }`), según su versión. Elige una forma y úsala en todo el proyecto.

**Mensajes constantes, datos en campos.** `"Sincronización terminada"` es siempre el mismo texto: puedes contar cuántas hubo. Si el mensaje fuera `"Sincronización de a terminada con 120"`, cada línea sería distinta.

## Excepciones: loguear una sola vez

```kotlin
try {
  sincronizar(proveedor)
} catch (e: CancellationException) {
  throw e
} catch (e: Exception) {
  logger.error(e) { "Falló la sincronización" }   // la excepción como primer argumento: incluye el stack trace
}
```

El antipatrón es **loguear y relanzar**:

```kotlin
catch (e: Exception) {
  logger.error(e) { "Error al buscar el pedido" }
  throw e                                         // ❌ quien la atrape la va a loguear de nuevo
}
```

El mismo error aparece tres o cuatro veces en los logs, cada vez con un mensaje distinto. Regla: **se loguea donde se maneja**. Si solo la relanzas, no la loguees. En una ruta, el lugar natural es el handler de StatusPages, como viste en [Manejo de errores con StatusPages](../14-ktor-y-koin/02-manejo-de-errores-con-statuspages.md).

## Lo que nunca va a un log

Los logs se copian a muchos lugares, los ve mucha gente y se guardan meses. Nunca:

* **Contraseñas, tokens, API keys, secretos de cifrado.** Ni completos ni "los primeros caracteres".
* **Headers `Authorization` y `Cookie`**, ni cuerpos de peticiones de login.
* **Presigned URLs**: son credenciales temporales ([Presigned URLs](../16-seguridad/08-presigned-urls.md)).
* **Datos personales** que no hagan falta: usa IDs (`clienteId`) en vez de emails o nombres.

Cuidado especial con el logging de los clientes HTTP y con hacer `toString()` de un objeto completo: una data class con un campo `password` lo imprime entero. Si una clase guarda un secreto, sobrescribe su `toString()` para ocultarlo.

-----

## Resumen en 5 líneas

1. Un log estructurado es JSON con campos: se busca, filtra y cuenta en lugar de leerse línea por línea.
2. kotlin-logging sobre SLF4J y Logback; el mensaje en lambda solo se evalúa si el nivel está activo.
3. `ERROR` es "alguien tiene que actuar"; logs a la consola, en JSON en los entornos desplegados, con las librerías ruidosas en `WARN`.
4. Mensajes constantes con los datos en campos; las excepciones se loguean una vez, donde se manejan.
5. Nunca loguees secretos, tokens, presigned URLs ni datos personales innecesarios.

-----

## Siguiente lección

[2. Correlation ID](02-correlation-id.md): unir todos los logs de una misma petición.
