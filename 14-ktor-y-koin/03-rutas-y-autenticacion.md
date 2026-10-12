# Rutas y autenticación por ruta

## En una frase

Las rutas son el adapter primario HTTP: se organizan como extensiones de `Route` agrupadas por contexto, viven en el composition root, y **cada una declara de forma explícita** cómo se autentica, incluidas las públicas.

-----

## Antes de empezar

Conviene que ya sepas:

* Errores y formato de respuesta: [Manejo de errores con StatusPages](02-manejo-de-errores-con-statuspages.md).
* Autenticación con JWT en Ktor: [JWT con RS256](../16-seguridad/02-jwt-rs256.md).
* El contexto de autorización: [RBAC y multi-tenancy](../16-seguridad/05-rbac-y-multitenancy.md).

Palabras nuevas (también están en el [Glosario](Glosario.md)):

* **Ruta:** La combinación de método HTTP y path que atiende un handler (`POST /api/v1/pedidos`).
* **Handler:** La lambda que se ejecuta cuando llega una petición a esa ruta.
* **Parámetro de path:** Un valor dentro de la URL, como el `{id}` de `/pedidos/{id}`.
* **Query parameter:** Un valor después del `?`, como `?pagina=2`.
* **Principal:** El objeto que representa a quien se autenticó en la petición.

-----

## Organizar las rutas

Cada grupo de rutas es una extensión de `Route` en su propio archivo:

```kotlin
// pedidos/infrastructure/web/PedidoRoutes.kt
fun Route.pedidoRoutes(
  crearPedido: CrearPedido,
  cancelarPedido: CancelarPedido,
  buscarPedido: BuscarPedido,
) {
  route("/pedidos") {
    post { /* crear */ }
    get("/{id}") { /* buscar */ }
    post("/{id}/cancelar") { /* cancelar */ }
  }
}
```

Y el composition root las monta todas:

```kotlin
fun Application.configurarRutas() {
  routing {
    route("/api/v1") {
      authenticate("usuario") {
        pedidoRoutes(get(), get(), get())
        facturaRoutes(get(), get())
      }
      authenticate("servicio") {
        integracionRoutes(get())
      }
    }
    publico {
      get("/health") { call.respond(HttpStatusCode.OK) }
      authRoutes(get())  // login y refresh
    }
  }
}
```

Mirando este único archivo ves **todas** las rutas de la API y cómo se autentica cada grupo.

## Versionado

Todas las rutas de negocio van bajo `/api/v1`. Cuando haga falta un cambio incompatible, se crea `/api/v2` para esos endpoints y `/api/v1` sigue funcionando hasta que los clientes migren. Esto es especialmente importante con apps móviles, porque no puedes obligar a todos a actualizar.

## Leer la petición

```kotlin
get("/{id}") {
  val id = call.parameters["id"]
    ?.let(::parsearPedidoId)
    ?: return@get call.respondError(HttpStatusCode.BadRequest, "ID_INVALIDO", "El id no es válido")

  val pagina = call.request.queryParameters["pagina"]?.toIntOrNull() ?: 1

  // ...
}

post {
  val request = call.receive<CrearPedidoRequest>()   // JSON -> DTO (lanza BadRequestException si falla)
  // ...
}
```

Fíjate en el `?.let ... ?: return@get`: el parámetro se valida y se convierte al tipo de dominio en la entrada, sin `!!`.

Para más seguridad de tipos, Ktor tiene el plugin **Resources**, que define las rutas como clases:

```kotlin
@Resource("/pedidos/{id}")
class PedidoPorId(val id: String)

get<PedidoPorId> { recurso -> /* recurso.id ya está extraído */ }
```

## Una ruta completa

```kotlin
post {
  val request = call.receive<CrearPedidoRequest>()
  val contexto = call.authContext()

  when (val outcome = crearPedido.ejecutar(request.toCommand(contexto))) {
    is CrearPedidoOutcome.Creado ->
      call.respond(HttpStatusCode.Created, outcome.pedido.toResponse())
    is CrearPedidoOutcome.Denegado ->
      call.respondError(HttpStatusCode.Forbidden, "SIN_PERMISO", "No tienes permiso para crear pedidos")
    CrearPedidoOutcome.SinStock ->
      call.respondError(HttpStatusCode.Conflict, "SIN_STOCK", "No hay stock suficiente")
  }
}
```

El patrón se repite en todas las rutas:

1. **Leer** la petición y convertirla a un command (`toCommand`).
2. **Llamar** al puerto de entrada.
3. **Traducir** el outcome a HTTP.

La ruta no tiene lógica de negocio. Si encuentras un `if` de negocio en una ruta, va en el use case.

## Obtener al usuario sin `!!`

```kotlin
fun ApplicationCall.usuarioId(): UsuarioId =
  principal<JWTPrincipal>()?.subject?.let(::UsuarioId)
    ?: throw NoAutenticadoException()

suspend fun ApplicationCall.authContext(): AuthContext {
  val usuario = usuarioId()
  val empresa = parameters["empresaId"]?.let(::EmpresaId) ?: throw NoAutenticadoException()
  return resolverContexto(usuario, empresa)   // busca asignaciones y permisos
}
```

`NoAutenticadoException` la convierte StatusPages en un `401`. Así ninguna ruta usa `principal<JWTPrincipal>()!!`.

## Autenticación explícita: ninguna ruta "por omisión"

El riesgo: alguien agrega `get("/pedidos/exportar")` **fuera** del bloque `authenticate` y, sin darse cuenta, la exportación queda pública.

La defensa es una convención: **toda ruta está dentro de exactamente un bloque que declara su autenticación**. Para las públicas también, con un marcador explícito:

```kotlin
/** Marca rutas intencionalmente públicas. No hace nada en tiempo de ejecución: documenta la decisión. */
fun Route.publico(build: Route.() -> Unit) = build()
```

Así, "olvidarse" de la autenticación se ve raro en el code review: una ruta fuera de `authenticate` y fuera de `publico` salta a la vista.

Y para no depender solo del code review, un **test de contrato** levanta la app, recorre las rutas registradas y verifica que cada una esté en una lista conocida de públicas o requiera autenticación. Viste la idea en [Tests de arquitectura](../12-arquitectura/06-tests-de-arquitectura.md).

## DTOs: `Request` y `Response`

* Lo que entra se llama `*Request`; lo que sale, `*Response`.
* Viven en el paquete `web` del adapter, con `@Serializable`.
* **Nunca** devuelvas una entidad de dominio directamente: si mañana le agregas un campo interno, aparece en la API sin que nadie lo decida.

```kotlin
@Serializable
data class PedidoResponse(val id: String, val estado: String, val total: String)

fun Pedido.toResponse() = PedidoResponse(id.value.toString(), estado.name, total.toPlainString())
```

-----

## Resumen en 5 líneas

1. Las rutas son extensiones de `Route` por contexto, y el composition root las monta todas en un solo lugar.
2. Todo va bajo `/api/v1`; los cambios incompatibles van en una versión nueva.
3. Cada ruta lee la petición y la convierte en un command, llama al puerto y traduce el outcome; sin lógica de negocio.
4. Toda ruta está en un bloque `authenticate` o en un `publico` explícito, y un test de contrato lo verifica.
5. El principal se obtiene con un helper sin `!!`; lo que entra es `*Request`, lo que sale es `*Response`, nunca una entidad.

-----

## Siguiente lección

[4. Inyección de dependencias con Koin](04-koin.md): de dónde salen los `get()` de las rutas.
