# PostgreSQL y HikariCP

## En una frase

PostgreSQL es la base relacional; HikariCP es un **pool de conexiones** que mantiene unas pocas conexiones abiertas y las presta a quien las necesita, porque abrir una conexión nueva en cada consulta es lento.

-----

## Antes de empezar

Conviene que ya sepas:

* Tablas y transacciones: [Transacciones y aislamiento](02-transacciones-y-aislamiento.md).
* El dispatcher de IO: [Scopes y dispatchers](../10-corrutinas/02-scopes-y-dispatchers.md).

Palabras nuevas (también están en el [Glosario](Glosario.md)):

* **JDBC:** La API estándar de la JVM para hablar con bases de datos. Es bloqueante.
* **Conexión:** Un canal abierto entre tu app y la base. Abrirla cuesta: red, autenticación y memoria en el servidor.
* **Pool de conexiones:** Conjunto de conexiones abiertas que se reutilizan.
* **`DataSource`:** La interfaz de JDBC que entrega conexiones; HikariCP la implementa.
* **Schema (Postgres):** Un espacio de nombres dentro de una base: agrupa tablas, como una carpeta.

-----

## Tipos de PostgreSQL que conviene usar

| Para | Usa | Evita |
| --- | --- | --- |
| Fechas y horas | `timestamptz` | `timestamp` sin zona horaria |
| Identificadores | `uuid` | IDs secuenciales expuestos en la API, si no quieres que se puedan adivinar |
| Texto | `text` | `varchar(255)` "por costumbre" |
| Dinero | `numeric(12, 2)` | `double precision` (pierde precisión) |
| Datos semiestructurados | `jsonb` | `json` (no se puede indexar igual) |

Sobre `timestamptz`: a pesar del nombre, **no guarda la zona horaria**. Guarda un instante en UTC y lo convierte al mostrarlo. Es exactamente lo que quieres: un instante sin ambigüedad, que en Kotlin se mapea a un `Instant`. Con `timestamp` a secas, `2026-03-01 10:00` no dice de qué país es.

## Schemas: carpetas dentro de la base

Una base de Postgres puede tener varios schemas. Por defecto todo va a `public`, pero puedes separar:

```sql
CREATE SCHEMA auth;
CREATE TABLE auth.usuarios (...);
CREATE TABLE public.pedidos (...);
```

¿Para qué? En un monolito modular, cada bounded context puede tener **su propio schema**, e incluso su propio usuario de base de datos con permisos solo sobre ese schema. Así, aunque compartan servidor, un módulo no puede leer las tablas de otro ni por accidente.

## El problema: abrir conexiones es caro

Abrir una conexión a Postgres implica conexión TCP, handshake TLS, autenticación y un proceso nuevo del lado del servidor. Son milisegundos, y además Postgres admite un número limitado de conexiones (por defecto `max_connections = 100`).

Si cada petición HTTP abre y cierra su propia conexión, la base se satura.

## HikariCP

```kotlin
fun crearDataSource(config: DbConfig): HikariDataSource {
  val hikari = HikariConfig().apply {
    jdbcUrl = config.url                 // jdbc:postgresql://localhost:5432/tienda
    username = config.usuario
    password = config.password
    maximumPoolSize = 10
    minimumIdle = 2
    connectionTimeout = 5_000            // ms esperando una conexión libre
    maxLifetime = 30 * 60_000            // ms antes de reciclar una conexión
    isAutoCommit = false                 // las transacciones las maneja Exposed
  }
  return HikariDataSource(hikari)
}
```

| Parámetro | Qué controla |
| --- | --- |
| `maximumPoolSize` | Cuántas conexiones como máximo |
| `minimumIdle` | Cuántas mantener abiertas aunque no se usen |
| `connectionTimeout` | Cuánto espera una petición por una conexión libre antes de fallar |
| `maxLifetime` | Cada cuánto se recicla una conexión (debe ser menor que los timeouts de la red o de la base) |

## ¿Cuántas conexiones? Menos de las que crees

La intuición dice "más conexiones = más rápido". Es falso. Postgres no ejecuta 200 consultas en paralelo de forma eficiente con 8 núcleos: las pone a competir por CPU y disco.

La guía de HikariCP propone como punto de partida:

```
conexiones = (núcleos de la base × 2) + discos efectivos
```

Para una base de 4 núcleos con SSD, alrededor de 10. Y recuerda que el total se multiplica por instancias: 3 instancias × 10 = 30 conexiones contra la base.

Si las peticiones esperan mucho por una conexión, el problema casi nunca es el tamaño del pool: son **transacciones largas** (lección 2) que retienen la conexión.

## Un pool por módulo

Si un módulo crítico (por ejemplo, autenticación) comparte pool con el resto, un reporte pesado puede tomar todas las conexiones y **dejar a nadie poder hacer login**.

Separar pools aísla la carga:

```kotlin
val dataSourcePrincipal = crearDataSource(config.principal)   // schema public
val dataSourceAuth = crearDataSource(config.auth)             // schema auth, otro usuario
```

Cada uno tiene su propio máximo, y uno saturado no afecta al otro. HikariCP permite fijar el schema por defecto de las conexiones con la propiedad `schema`.

## JDBC bloquea: dónde correrlo

Cada consulta JDBC bloquea el hilo hasta que Postgres responde. En un servidor con corrutinas, eso tiene que correr en `Dispatchers.IO` (lo viste en [Scopes y dispatchers](../10-corrutinas/02-scopes-y-dispatchers.md)). Una regla útil: el pool de IO tiene que poder cubrir, como mínimo, la suma de los `maximumPoolSize` de todos tus pools.

-----

## Resumen en 5 líneas

1. Usa `timestamptz` para instantes, `numeric` para dinero, `uuid` para IDs públicos y `jsonb` para datos semiestructurados.
2. Los schemas de Postgres separan tablas por módulo, y se pueden combinar con usuarios de permisos limitados.
3. Abrir conexiones es caro; HikariCP mantiene un pool y las reutiliza.
4. El pool rinde más chico de lo que parece: empieza por núcleos × 2 y multiplica por instancias.
5. Un pool por módulo crítico aísla la carga; JDBC bloquea, así que corre en `Dispatchers.IO`.

-----

## Siguiente lección

[4. Exposed DSL](04-exposed-dsl.md): escribir las consultas en Kotlin.
