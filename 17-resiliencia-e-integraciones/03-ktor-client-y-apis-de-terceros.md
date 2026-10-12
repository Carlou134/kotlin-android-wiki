# Ktor Client y APIs de terceros

## En una frase

Integrar una API de terceros es leer bien su documentación, configurar **un** cliente HTTP por proveedor (timeouts, autenticación, reintentos), traducir sus modelos a los tuyos en una Anti-Corruption Layer, y testear todo sin llamar al proveedor real.

-----

## Antes de empezar

Conviene que ya sepas:

* Reintentos e idempotencia: [Reintentos con backoff](01-reintentos-con-backoff.md) y [Idempotencia](02-idempotencia.md).
* Qué es una Anti-Corruption Layer: [DDD y monolito modular](../12-arquitectura/03-ddd-y-monolito-modular.md).
* Outcomes con `sealed`: [Clases e interfaces selladas](../08-poo-avanzada/02-sealed.md).

Palabras nuevas (también están en el [Glosario](Glosario.md)):

* **Ktor Client:** El cliente HTTP de Ktor, multiplataforma y basado en corrutinas.
* **Engine (cliente):** La librería HTTP que usa Ktor Client por debajo: OkHttp, CIO, Java, etc.
* **CookieJar:** Componente de OkHttp que guarda y reenvía cookies entre peticiones.
* **Paginación por cursor:** El proveedor devuelve un marcador (`cursor`) para pedir la página siguiente.
* **`MockEngine`:** Engine falso de Ktor Client que responde lo que tú le digas, para tests.

-----

## Antes del código: leer la API del proveedor

La mitad del trabajo de una integración es **entender** al proveedor. Una lista para leer su documentación:

| Pregunta | Por qué importa |
| --- | --- |
| ¿Cómo se autentica? (API key, OAuth, login con cookies) | Define cómo se arma el cliente |
| ¿Cuánto duran los tokens y cómo se renuevan? | Para no fallar a mitad de una sincronización |
| ¿Qué límites de peticiones tiene? | Para no recibir `429` en cada ejecución |
| ¿Cómo pagina? (por número de página, por cursor) | Para no perder ni repetir registros |
| ¿Qué formato tienen los errores? | Para clasificarlos en transitorios y permanentes |
| ¿En qué zona horaria y unidades vienen los datos? | `1730000000` ¿son segundos o milisegundos? ¿La batería es 0–1 o 0–100? |
| ¿Hay sandbox o entorno de pruebas? | Para probar sin afectar datos reales |
| ¿Cómo versiona la API? | Para saber qué puede romperse sin aviso |

Y una regla: **no confíes en la documentación**. Guarda ejemplos reales de respuestas (sin datos sensibles) y úsalos en los tests. Es muy común que la API real no coincida con lo documentado.

## Un cliente por proveedor

```kotlin
fun crearClienteProveedorA(config: ProveedorAConfig): HttpClient = HttpClient(OkHttp) {
  expectSuccess = false   // manejamos los códigos de estado nosotros

  install(ContentNegotiation) {
    json(Json { ignoreUnknownKeys = true })   // el proveedor agrega campos sin avisar
  }

  install(HttpTimeout) {
    connectTimeoutMillis = 5_000
    requestTimeoutMillis = 30_000
    socketTimeoutMillis = 15_000
  }

  install(HttpRequestRetry) {
    retryOnExceptionOrServerErrors(maxRetries = 3)
    exponentialDelay()
  }

  defaultRequest {
    url(config.baseUrl)          // https://api.proveedor-a.com/v2/
    header("X-Api-Key", config.apiKey)
  }
}
```

Puntos clave:

* **Reutiliza el cliente.** Crear un `HttpClient` por petición abre conexiones y pools nuevos cada vez. Se crea **uno** por proveedor (un `single` de Koin) y se cierra al apagar la app.
* **`ignoreUnknownKeys = true`:** si el proveedor agrega un campo nuevo, tu integración no se rompe.
* **Timeouts siempre.** Sin timeouts, una conexión colgada bloquea la sincronización para siempre.
* **Logging con cuidado.** El plugin `Logging` puede imprimir headers y cuerpos: filtra las API keys, los tokens y las contraseñas antes de activarlo.

## Autenticación con tokens que vencen

Si el proveedor usa tokens que vencen, el plugin `Auth` de Ktor los maneja:

```kotlin
install(Auth) {
  bearer {
    loadTokens { tokenStore.actual() }
    refreshTokens {                       // se llama al recibir un 401
      val nuevo = loginProveedor(config)  // pide un token nuevo
      tokenStore.guardar(nuevo)
      nuevo
    }
  }
}
```

Al recibir un `401`, el plugin renueva el token **una vez** y repite la petición. Es el caso del `401` que sí vale la pena reintentar.

## Proveedores que usan cookies de sesión

Algunas APIs, sobre todo las viejas, no usan tokens: haces login y el servidor te da una **cookie de sesión** que hay que reenviar en cada petición. Con el engine OkHttp, eso lo resuelve un `CookieJar`:

```kotlin
HttpClient(OkHttp) {
  engine {
    config {
      cookieJar(CookieJarEnMemoria())   // guarda las cookies que manda el servidor y las reenvía
    }
  }
}
```

Ktor también tiene un plugin propio, `HttpCookies`, que hace lo mismo sin depender del engine.

## Llamar y clasificar la respuesta

```kotlin
sealed interface ResultadoProveedor<out T> {
  data class Ok<T>(val valor: T) : ResultadoProveedor<T>
  data object NoEncontrado : ResultadoProveedor<Nothing>
  data class ErrorTransitorio(val detalle: String) : ResultadoProveedor<Nothing>
  data class ErrorPermanente(val detalle: String) : ResultadoProveedor<Nothing>
}

suspend fun obtenerDispositivo(id: String): ResultadoProveedor<Dispositivo> {
  val respuesta = cliente.get("devices/$id")
  return when {
    respuesta.status.isSuccess() ->
      ResultadoProveedor.Ok(respuesta.body<DeviceDto>().toDomain())
    respuesta.status == HttpStatusCode.NotFound ->
      ResultadoProveedor.NoEncontrado
    respuesta.status.value == 429 || respuesta.status.value >= 500 ->
      ResultadoProveedor.ErrorTransitorio("HTTP ${respuesta.status.value}")
    else ->
      ResultadoProveedor.ErrorPermanente("HTTP ${respuesta.status.value}: ${respuesta.bodyAsText().take(500)}")
  }
}
```

El código de estado del proveedor **no** sale del adapter. El resto del sistema ve un resultado en su propio idioma.

## Paginación

Por cursor, que es lo más común en APIs modernas:

```kotlin
suspend fun obtenerTodos(): List<Dispositivo> {
  val todos = mutableListOf<Dispositivo>()
  var cursor: String? = null
  do {
    val pagina = cliente.get("devices") {
      parameter("limit", 100)
      cursor?.let { parameter("cursor", it) }
    }.body<PaginaDto<DeviceDto>>()

    todos += pagina.items.map { it.toDomain() }
    cursor = pagina.siguienteCursor
  } while (cursor != null)
  return todos
}
```

Con miles de registros, no acumules todo en memoria: procesa página por página (o devuelve un `Flow` que emita cada página).

## La Anti-Corruption Layer

El DTO del proveedor vive **solo** en el adapter, y se traduce al dominio con una extensión:

```kotlin
@Serializable
private data class DeviceDto(
  val sn: String,
  val batt: Double,       // 0.0 a 1.0 en este proveedor
  val st: Int,            // 0 = offline, 1 = idle, 2 = working
  val ts: Long,           // epoch en SEGUNDOS
)

private fun DeviceDto.toDomain() = Dispositivo(
  idExterno = sn,
  bateria = (batt * 100).roundToInt(),                 // tu dominio usa 0..100
  estado = when (st) {
    0 -> Estado.DESCONECTADO
    1 -> Estado.INACTIVO
    2 -> Estado.TRABAJANDO
    else -> Estado.DESCONOCIDO                          // no rompas por un código nuevo
  },
  ultimaSenal = Instant.fromEpochSeconds(ts),
)
```

Con varios proveedores, cada uno tiene su DTO y su `toDomain()`, y el dominio ve un solo modelo `Dispositivo`. Las rarezas de cada proveedor (unidades, códigos, zonas horarias) quedan **encerradas** en su adapter.

## Testear sin llamar al proveedor

`MockEngine` reemplaza la red por respuestas que tú defines:

```kotlin
test("traduce la batería de 0..1 a 0..100") {
  val engine = MockEngine { _ ->
    respond(
      content = """{"sn":"R-1","batt":0.87,"st":2,"ts":1760280000}""",
      status = HttpStatusCode.OK,
      headers = headersOf(HttpHeaders.ContentType, "application/json"),
    )
  }
  val adapter = ProveedorAAdapter(HttpClient(engine) { /* misma config */ })

  val resultado = adapter.obtenerDispositivo("R-1")

  resultado shouldBe ResultadoProveedor.Ok(Dispositivo("R-1", 87, Estado.TRABAJANDO, ...))
}
```

Usa como `content` las respuestas reales que guardaste. Prueba también los casos feos: `429`, `503`, JSON con campos de más, códigos de estado desconocidos.

-----

## Resumen en 5 líneas

1. Lee la API del proveedor con una lista: autenticación, límites, paginación, errores, unidades y zonas horarias; y desconfía de la documentación.
2. Un `HttpClient` por proveedor, reutilizado, con timeouts, `ignoreUnknownKeys`, reintentos y sin loguear secretos.
3. Tokens que vencen con el plugin `Auth`; sesiones con cookies con un `CookieJar` de OkHttp o el plugin `HttpCookies`.
4. Clasifica la respuesta en un resultado `sealed` y traduce el DTO a tu dominio en la Anti-Corruption Layer.
5. Testea con `MockEngine` usando respuestas reales, incluidos los errores.

-----

## Siguiente lección

[4. Redis](04-redis.md): memoria compartida y rápida.
