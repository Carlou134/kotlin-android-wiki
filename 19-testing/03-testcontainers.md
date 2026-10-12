# Testcontainers

## En una frase

Testcontainers levanta un PostgreSQL (o Redis, o MinIO) **real** en un contenedor de Docker durante los tests y lo destruye al terminar; así los repositorios se prueban contra la misma tecnología que usa producción, con las mismas migraciones.

-----

## Antes de empezar

Conviene que ya sepas:

* Exposed, Flyway y el pool de conexiones: [Exposed DSL](../15-persistencia/04-exposed-dsl.md), [Migraciones con Flyway](../15-persistencia/05-migraciones-con-flyway.md) y [PostgreSQL y HikariCP](../15-persistencia/03-postgresql-y-hikaricp.md).
* Cuándo no mockear: [MockK](02-mockk.md).

Palabras nuevas (también están en el [Glosario](Glosario.md)):

* **Contenedor:** Un proceso aislado con su propio sistema de archivos, creado a partir de una imagen de Docker.
* **Testcontainers:** Librería que crea y destruye contenedores desde el código de los tests.
* **Base en memoria (H2):** Base de datos que corre dentro de la JVM; rápida, pero no es PostgreSQL.
* **Test de integración:** Test que prueba una pieza junto con tecnología real.

-----

## ¿Por qué no una base en memoria?

H2 es rápida y no necesita Docker. Pero **no es PostgreSQL**:

| Lo que usas | ¿Funciona igual en H2? |
| --- | --- |
| `INSERT ... ON CONFLICT DO UPDATE` | No, o con otra sintaxis |
| `jsonb`, `timestamptz` | No exactamente |
| `SELECT ... FOR UPDATE SKIP LOCKED` | No |
| Índices parciales, `CREATE INDEX CONCURRENTLY` | No |
| Tus migraciones de Flyway | Muchas fallan |

Un test que pasa en H2 te dice que tu código funciona **en H2**. Tus repositorios usan justamente lo que H2 no tiene. Por eso los tests de repositorios corren contra PostgreSQL real.

## El contenedor compartido

Levantar un contenedor tarda unos segundos. Hacerlo en **cada** test sería lentísimo. La práctica es levantarlo **una vez** para toda la ejecución:

```kotlin
object PostgresDePrueba {

  private val contenedor = PostgreSQLContainer("postgres:16-alpine").apply { start() }

  val dataSource: HikariDataSource by lazy {
    HikariDataSource(HikariConfig().apply {
      jdbcUrl = contenedor.jdbcUrl
      username = contenedor.username
      password = contenedor.password
      maximumPoolSize = 5
    }).also { migrar(it) }   // las MISMAS migraciones de Flyway que producción
  }

  val database: Database by lazy { Database.connect(dataSource) }
}
```

* **Usa la misma versión mayor de PostgreSQL que producción.** Si producción usa 16, el test usa 16.
* **Corre las migraciones de Flyway**, no `SchemaUtils.create`. Así cada test también verifica que las migraciones funcionan.
* El `object` se inicializa la primera vez que un test lo usa y vive hasta que termina la JVM. Testcontainers destruye el contenedor al final (con un contenedor auxiliar llamado Ryuk).

> El paquete de la clase `PostgreSQLContainer` cambió entre versiones mayores de Testcontainers: revisa el import según la versión que uses.

## Limpiar entre tests

Con una base compartida, los datos de un test aparecen en el siguiente. La forma más simple:

```kotlin
class ExposedPedidoRepositoryIntegrationTest : FunSpec({

  val repo = ExposedPedidoRepository(PostgresDePrueba.database)

  beforeTest {
    transaction(PostgresDePrueba.database) {
      exec("TRUNCATE pedidos, lineas_pedido RESTART IDENTITY CASCADE")
    }
  }

  test("guarda y recupera un pedido") {
    val pedido = unPedido()
    repo.guardar(pedido)

    repo.buscar(pedido.id) shouldBe pedido
  }

  test("buscar un id inexistente devuelve null") {
    repo.buscar(PedidoId(UUID.randomUUID().toString())).shouldBeNull()
  }
})
```

Una convención útil: los tests que usan contenedores terminan en `IntegrationTest`. Así se distinguen a simple vista y se pueden correr por separado en Gradle.

## Qué testear en un repositorio

No testees que Exposed sabe hacer un `INSERT`. Testea **lo tuyo**:

* **El mapeo:** lo que se guarda vuelve igual, con todos los campos, incluidas fechas y decimales.
* **Las consultas con filtros:** paginación, orden, filtros combinados.
* **El aislamiento por tenant:** crea datos de dos empresas y verifica que una no ve los de la otra.
* **Los constraints:** un duplicado viola el `UNIQUE` y el repositorio lo traduce bien.
* **La concurrencia:** que el compare-and-swap solo deja ganar a uno.

## Probar concurrencia de verdad

Este tipo de test **solo** es posible contra una base real:

```kotlin
test("dos consumos simultáneos del mismo token: solo uno gana") {
  val token = tokens.crear(unRefreshToken())

  val resultados = coroutineScope {
    (1..2).map {
      async(Dispatchers.IO) { tokens.consumir(token.hash) }
    }.awaitAll()
  }

  resultados.count { it == ConsumoResultado.Consumido } shouldBe 1
  resultados.count { it == ConsumoResultado.YaUsado } shouldBe 1
}
```

Es la prueba concreta del compare-and-swap de [Bloqueos y concurrencia](../15-persistencia/06-bloqueos-y-concurrencia.md) y de la rotación de [Refresh tokens](../16-seguridad/04-refresh-tokens.md).

## Otros contenedores

El mismo enfoque sirve para todo lo que corre en Docker:

```kotlin
val redis = GenericContainer("redis:7-alpine").withExposedPorts(6379).apply { start() }
val urlRedis = "redis://${redis.host}:${redis.getMappedPort(6379)}"
```

Testcontainers asigna un **puerto aleatorio** del lado de tu máquina, así varios tests (o varios proyectos) no chocan. Por eso siempre se pide con `getMappedPort`.

## Requisitos y costos

* Necesita **Docker** corriendo, en tu máquina y en el CI.
* El primer arranque descarga la imagen; después queda en caché.
* Son más lentos que los unitarios: por eso son la **mitad** de la pirámide, no la base.

-----

## Resumen en 5 líneas

1. Testea los repositorios contra PostgreSQL real: H2 no tiene `ON CONFLICT`, `jsonb`, `SKIP LOCKED` ni corre tus migraciones.
2. Levanta un contenedor compartido una sola vez, con la misma versión que producción y las migraciones de Flyway.
3. Limpia los datos entre tests (`TRUNCATE`) y nombra estos tests `*IntegrationTest`.
4. Testea mapeo, filtros, aislamiento por tenant, constraints y concurrencia, no que Exposed sabe insertar.
5. Sirve para Redis, MinIO o cualquier imagen; usa siempre el puerto mapeado, y requiere Docker.

-----

## Siguiente lección

[4. `testApplication`](04-test-application.md): probar la API completa.
