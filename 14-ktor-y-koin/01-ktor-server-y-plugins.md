# Ktor server y plugins

## En una frase

Una aplicación Ktor es un **pipeline**: cada petición pasa por una cadena de plugins (serialización, logging, autenticación, manejo de errores) antes y después de llegar a tu ruta; tú eliges qué plugins instalar y cómo configurarlos.

-----

## Antes de empezar

Conviene que ya sepas:

* Qué es el composition root: [Composition root](../12-arquitectura/04-composition-root.md).
* Que cada petición es una corrutina: [Scopes y dispatchers](../10-corrutinas/02-scopes-y-dispatchers.md).

Palabras nuevas (también están en el [Glosario](Glosario.md)):

* **Engine:** El servidor HTTP de bajo nivel que usa Ktor (Netty, CIO, Jetty).
* **Módulo de Ktor:** Una función `Application.module()` que configura la aplicación. No confundir con un módulo de Gradle o de Koin.
* **Plugin:** Componente que se instala en el pipeline y agrega comportamiento a todas las peticiones (antes se llamaban *features*).
* **Pipeline:** La cadena de fases por la que pasa cada petición y su respuesta.
* **`ApplicationCall`:** El objeto `call`: la petición y su respuesta en curso.

-----

## Ktor es una librería, no un framework

Ktor no escanea clases ni usa anotaciones mágicas. **Tú** llamas a todo de forma explícita: instalas plugins, registras rutas, pasas dependencias. Lo que no instalas, no existe.

Eso es justo lo que encaja con una arquitectura hexagonal: Ktor queda en el borde, y el núcleo no sabe que existe.

## Arrancar el servidor

Hay dos formas.

**`embeddedServer`:** todo en código. Simple, ideal para empezar.

```kotlin
fun main() {
  embeddedServer(Netty, port = 8080, module = Application::module).start(wait = true)
}
```

**`EngineMain`:** la configuración vive en un archivo (`application.yaml` o `application.conf`).

```kotlin
fun main(args: Array<String>) = EngineMain.main(args)
```

```yaml
# src/main/resources/application.yaml
ktor:
  deployment:
    port: 8080
  application:
    modules:
      - com.tienda.app.ApplicationKt.module

db:
  url: "$DB_URL:jdbc:postgresql://localhost:5432/tienda"
  usuario: "$DB_USER:tienda"
```

La sintaxis `"$VARIABLE:valor"` lee una variable de entorno y usa el valor por defecto si no existe. Así el mismo artefacto sirve para local, staging y producción. Para leerla:

```kotlin
val dbUrl = environment.config.property("db.url").getString()
```

## El módulo: el composition root en Ktor

```kotlin
fun Application.module() {
  configurarDependencias()   // Koin (lección 4)
  configurarSerializacion()
  configurarObservabilidad()
  configurarCors()
  configurarSeguridad()      // Authentication con JWT
  configurarErrores()        // StatusPages (lección 2)
  configurarRutas()          // lección 3
  configurarJobs()           // jobs programados
}
```

Cada `configurarX()` es una extensión de `Application` en su propio archivo. El módulo se lee como un índice de todo lo que tiene la aplicación.

## Los plugins esenciales

### ContentNegotiation: JSON de ida y vuelta

```kotlin
fun Application.configurarSerializacion() {
  install(ContentNegotiation) {
    json(Json {
      ignoreUnknownKeys = true   // un campo extra del cliente no rompe la petición
      explicitNulls = false      // no manda "campo": null en las respuestas
    })
  }
}
```

Sin este plugin, `call.receive<CrearPedidoRequest>()` y `call.respond(pedido)` no saben convertir a JSON.

### CallId: un identificador por petición

```kotlin
install(CallId) {
  retrieveFromHeader(HttpHeaders.XRequestId)   // si el cliente o el proxy ya mandan uno, se reutiliza
  generate { UUID.randomUUID().toString() }    // si no, se genera
  replyToHeader(HttpHeaders.XRequestId)        // se devuelve en la respuesta
}
```

Con ese ID puedes seguir una petición por todos los logs, incluso entre servicios. Se profundiza en [Correlation ID](../18-observabilidad/02-correlation-id.md).

### CallLogging: una línea de log por petición

```kotlin
install(CallLogging) {
  level = Level.INFO
  callIdMdc("call-id")                                    // el CallId aparece en cada log
  filter { call -> call.request.path().startsWith("/api") } // sin ruido de health checks
}
```

### CORS: qué sitios web pueden llamar a tu API

Un navegador bloquea las peticiones de `app.tienda.com` a `api.tienda.com` a menos que la API lo permita explícitamente:

```kotlin
install(CORS) {
  allowHost("app.tienda.com", schemes = listOf("https"))
  allowHeader(HttpHeaders.Authorization)
  allowHeader(HttpHeaders.ContentType)
  allowMethod(HttpMethod.Put)
  allowMethod(HttpMethod.Delete)
}
```

Nunca uses `anyHost()` en producción para una API con autenticación. CORS se explica a fondo en [HTTP a fondo](../13-http-y-apis/01-http-a-fondo.md).

### Authentication

Lo viste en [JWT con RS256](../16-seguridad/02-jwt-rs256.md): `install(Authentication) { jwt("usuario") { ... } }`. Puedes registrar varios proveedores con nombre (por ejemplo, `"usuario"` y `"servicio"`), y cada ruta elige cuál acepta.

### Swagger UI

Si tienes un archivo OpenAPI, Ktor lo puede servir con una interfaz navegable:

```kotlin
routing {
  swaggerUI(path = "swagger", swaggerFile = "openapi/documentation.yaml")
}
```

## Plugins propios

Si necesitas algo transversal que no existe, lo creas:

```kotlin
val TiempoDeRespuesta = createApplicationPlugin(name = "TiempoDeRespuesta") {
  onCall { call ->
    call.attributes.put(InicioKey, TimeSource.Monotonic.markNow())
  }
  onCallRespond { call ->
    val inicio = call.attributes[InicioKey]
    call.response.header("X-Response-Time", "${inicio.elapsedNow().inWholeMilliseconds}ms")
  }
}

private val InicioKey = AttributeKey<TimeSource.Monotonic.ValueTimeMark>("inicio")

// install(TiempoDeRespuesta)
```

* `onCall` corre al llegar la petición; `onCallRespond`, al enviar la respuesta.
* `call.attributes` es un mapa tipado para pasar datos entre fases de la misma petición.

## Apagado ordenado

Al apagar el servidor, hay que cerrar lo que se abrió (por ejemplo, el pool de conexiones):

```kotlin
monitor.subscribe(ApplicationStopped) {
  dataSource.close()
}
```

Los jobs lanzados en el scope de la `Application` se cancelan solos, como viste en [Jobs programados](../10-corrutinas/04-jobs-programados.md).

-----

## Resumen en 5 líneas

1. Ktor es una librería explícita: lo que no instalas no existe, y eso deja al framework en el borde.
2. Se arranca con `embeddedServer` (todo en código) o `EngineMain` (configuración en `application.yaml`, con variables de entorno).
3. `Application.module()` es el composition root: llama a funciones `configurarX()`, una por tema.
4. Los plugins esenciales son ContentNegotiation, CallId, CallLogging, CORS, Authentication y StatusPages.
5. Con `createApplicationPlugin` creas plugins propios; con `monitor.subscribe(ApplicationStopped)` cierras recursos.

-----

## Siguiente lección

[2. Manejo de errores con StatusPages](02-manejo-de-errores-con-statuspages.md): qué ve el cliente cuando algo sale mal.
