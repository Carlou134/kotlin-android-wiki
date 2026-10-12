# Manejo de errores con StatusPages

## En una frase

StatusPages es el plugin que atrapa las excepciones que escapan de las rutas y las convierte en respuestas HTTP con el código correcto y un formato de error **único** para toda la API.

-----

## Antes de empezar

Conviene que ya sepas:

* Instalar plugins: [Ktor server y plugins](01-ktor-server-y-plugins.md).
* Outcomes y cuándo usar excepciones: [La capa application](../12-arquitectura/05-capa-application.md).

Palabras nuevas (también están en el [Glosario](Glosario.md)):

* **StatusPages:** Plugin de Ktor que mapea excepciones y códigos de estado a respuestas.
* **Formato de error:** La forma única que tienen todas las respuestas de error de la API.
* **Problem Details:** Estándar (RFC 9457) para el formato de errores en APIs HTTP.
* **RequestValidation:** Plugin de Ktor que valida el cuerpo de la petición antes de que llegue a la ruta.

-----

## El problema: cada error con su propia forma

Sin un manejo central:

* Una ruta responde `{"error": "no existe"}`, otra `{"message": "Not found"}` y otra texto plano.
* Una excepción no atrapada devuelve `500` con un stack trace que revela clases internas, versiones de librerías y a veces datos.
* El cliente (app móvil, frontend) no puede manejar los errores de forma genérica.

## Un formato de error para toda la API

```kotlin
@Serializable
data class ErrorResponse(
  val codigo: String,                       // estable, para que el cliente lo use en código
  val mensaje: String,                      // legible, para humanos
  val detalles: List<String> = emptyList(),
  val callId: String? = null,               // para buscar en los logs
)
```

```json
{
  "codigo": "PEDIDO_YA_ENVIADO",
  "mensaje": "El pedido ya fue enviado y no se puede cancelar",
  "callId": "8f3c2a..."
}
```

El `codigo` es el contrato: el cliente hace `if (codigo == "PEDIDO_YA_ENVIADO")`, nunca compara el mensaje. El `callId` permite que soporte encuentre la petición exacta en los logs.

Si prefieres un estándar, **Problem Details** (RFC 9457) define campos como `type`, `title`, `status` y `detail`. Lo importante no es cuál eliges, sino que sea **uno solo**.

## Instalar StatusPages

```kotlin
fun Application.configurarErrores() {
  install(StatusPages) {

    exception<RequestValidationException> { call, e ->
      call.respondError(HttpStatusCode.BadRequest, "VALIDACION", "La petición no es válida", e.reasons)
    }

    exception<BadRequestException> { call, _ ->
      call.respondError(HttpStatusCode.BadRequest, "PETICION_INVALIDA", "El cuerpo de la petición no es válido")
    }

    exception<NoAutenticadoException> { call, _ ->
      call.respondError(HttpStatusCode.Unauthorized, "NO_AUTENTICADO", "Se requiere autenticación")
    }

    exception<Throwable> { call, e ->
      call.application.log.error("Error no controlado", e)
      call.respondError(HttpStatusCode.InternalServerError, "ERROR_INTERNO", "Ocurrió un error inesperado")
    }

    status(HttpStatusCode.NotFound) { call, _ ->
      call.respondError(HttpStatusCode.NotFound, "NO_ENCONTRADO", "El recurso no existe")
    }
  }
}

private suspend fun ApplicationCall.respondError(
  status: HttpStatusCode,
  codigo: String,
  mensaje: String,
  detalles: List<String> = emptyList(),
) = respond(status, ErrorResponse(codigo, mensaje, detalles, callId))
```

Puntos clave:

* Ktor elige el handler de la excepción **más específica**. `Throwable` funciona como red de seguridad final.
* El handler de `Throwable` **registra** el error completo en el log, pero al cliente le manda un mensaje genérico. El stack trace nunca sale del servidor.
* `status(HttpStatusCode.NotFound)` también cubre las rutas que no existen, que de otro modo responderían vacío.
* Cuando `call.receive<T>()` no puede leer el JSON, Ktor lanza una `BadRequestException`. Sin un handler propio, igual responde `400`, pero sin tu formato de error.

## Outcomes vs excepciones en las rutas

En [La capa application](../12-arquitectura/05-capa-application.md) viste que los errores de negocio esperados son **outcomes**, no excepciones. Entonces, ¿quién convierte un outcome en HTTP?

```kotlin
post("/{id}/cancelar") {
  when (val outcome = cancelarPedido.ejecutar(command)) {
    is CancelarPedidoOutcome.Cancelado ->
      call.respond(HttpStatusCode.OK, outcome.pedido.toResponse())
    CancelarPedidoOutcome.NoEncontrado ->
      call.respondError(HttpStatusCode.NotFound, "PEDIDO_NO_ENCONTRADO", "El pedido no existe")
    is CancelarPedidoOutcome.YaEnviado ->
      call.respondError(HttpStatusCode.Conflict, "PEDIDO_YA_ENVIADO", "El pedido ya fue enviado")
    is CancelarPedidoOutcome.Denegado ->
      call.respondError(HttpStatusCode.Forbidden, "SIN_PERMISO", "No tienes permiso para esta acción")
  }
}
```

El reparto queda así:

| Tipo de error | Dónde se convierte en HTTP |
| --- | --- |
| Error de negocio esperado (outcome) | En la ruta, con un `when` exhaustivo |
| Petición mal formada | StatusPages (`BadRequestException`, `RequestValidationException`) |
| Problema transversal (no autenticado) | StatusPages, con una excepción propia |
| Fallo inesperado | StatusPages (`Throwable`), con log y `500` genérico |

Si te tienta lanzar una excepción desde el use case "para que StatusPages la maneje", piénsalo dos veces: estás escondiendo un error de negocio en un camino que el compilador no te obliga a manejar.

## Validar la entrada antes de la ruta

El plugin RequestValidation valida los DTOs al recibirlos:

```kotlin
install(RequestValidation) {
  validate<CrearPedidoRequest> { request ->
    when {
      request.lineas.isEmpty() -> ValidationResult.Invalid("El pedido debe tener al menos una línea")
      request.lineas.any { it.cantidad <= 0 } -> ValidationResult.Invalid("La cantidad debe ser mayor a cero")
      else -> ValidationResult.Valid
    }
  }
}
```

Si falla, lanza `RequestValidationException`, que StatusPages convierte en `400` con los motivos.

¿Esto reemplaza las validaciones del dominio? **No.** RequestValidation revisa la **forma** de la petición (campos vacíos, formatos). Las reglas de negocio ("un pedido no puede superar el crédito del cliente") viven en el dominio y en los use cases, porque valen sin importar por dónde entre la llamada.

-----

## Resumen en 5 líneas

1. Toda la API responde los errores con un formato único, con un `codigo` estable y el `callId`.
2. StatusPages mapea excepciones (la más específica gana) y códigos de estado a respuestas.
3. El handler de `Throwable` registra el error completo y responde un `500` genérico, sin stack traces.
4. Los outcomes de negocio se traducen en la ruta con `when`; StatusPages queda para lo transversal y lo inesperado.
5. RequestValidation valida la forma de la petición; las reglas de negocio siguen en el dominio.

-----

## Siguiente lección

[3. Rutas y autenticación por ruta](03-rutas-y-autenticacion.md): cómo organizar los endpoints.
