# Correlation ID

## En una frase

Un correlation ID es un identificador único que se asigna a cada petición (o a cada ejecución de un job) y aparece en **todos** sus logs, en la respuesta de error y en las llamadas a otros servicios; con él, encuentras la historia completa de un problema con una sola búsqueda.

-----

## Antes de empezar

Conviene que ya sepas:

* Los plugins CallId y CallLogging: [Ktor server y plugins](../14-ktor-y-koin/01-ktor-server-y-plugins.md).
* Logs estructurados: [Logging estructurado](01-logging-estructurado.md).
* Que una corrutina puede cambiar de hilo: [Scopes y dispatchers](../10-corrutinas/02-scopes-y-dispatchers.md).

Palabras nuevas (también están en el [Glosario](Glosario.md)):

* **Correlation ID:** Identificador que une todos los logs de una misma petición o ejecución.
* **MDC (Mapped Diagnostic Context):** Mapa de valores que Logback agrega automáticamente a cada log del hilo actual.
* **Thread-local:** Valor guardado por hilo: cada hilo ve el suyo.
* **Propagación:** Pasar el correlation ID a los servicios que llamas, para seguir la historia entre sistemas.
* **Traza distribuida:** El recorrido completo de una petición por varios servicios, con tiempos de cada paso.

-----

## El problema: 50 peticiones mezcladas

Un cliente reporta: "a las 14:30 me dio error al cancelar un pedido". En los logs, a las 14:30 hay 2.000 líneas de 50 peticiones simultáneas, de 3 instancias, todas intercaladas. ¿Cuáles son las de **esa** petición?

## La solución en tres pasos

**1. Asignar un ID a cada petición.** El plugin CallId lo toma del header `X-Request-Id` (si un proxy o el cliente ya lo mandan) o lo genera:

```kotlin
install(CallId) {
  retrieveFromHeader(HttpHeaders.XRequestId)
  generate { UUID.randomUUID().toString() }
  verify { it.length in 8..64 }               // no aceptar cualquier cosa del cliente
  replyToHeader(HttpHeaders.XRequestId)
}
```

**2. Ponerlo en cada log.** CallLogging lo copia al MDC:

```kotlin
install(CallLogging) {
  callIdMdc("callId")
}
```

Desde ese momento, **cada** log de esa petición, incluidos los de tus use cases y repositorios, lleva el campo `"callId":"8f3c2a10"`, sin que tengas que pasarlo a mano.

**3. Devolverlo al cliente.** En la respuesta va el header `X-Request-Id`, y en las respuestas de error, el campo `callId`, como viste en [Manejo de errores con StatusPages](../14-ktor-y-koin/02-manejo-de-errores-con-statuspages.md). Soporte le pide ese valor al cliente y busca `callId = 8f3c2a10`: aparece la historia completa de la petición, y nada más.

## MDC y corrutinas: la trampa

El MDC de SLF4J es **thread-local**: está atado al hilo. Pero una corrutina puede suspender en un hilo y retomar en **otro**, por ejemplo al pasar por `withContext(Dispatchers.IO)`. En el hilo nuevo, el MDC no tiene tu `callId`, y los logs salen sin él.

La librería `kotlinx-coroutines-slf4j` lo resuelve con `MDCContext`, un elemento de contexto de corrutina que **lleva el MDC con la corrutina** y lo restaura en cada hilo donde retoma:

```kotlin
suspend fun buscarPedido(id: PedidoId): Pedido? =
  withContext(Dispatchers.IO + MDCContext()) {   // captura el MDC actual y lo lleva al hilo de IO
    logger.debug { "Buscando pedido" }           // ✅ sale con callId
    // ...
  }
```

Dentro de las rutas, el plugin de Ktor ya maneja el MDC durante el pipeline. El cuidado es en **tu** código cuando cambias de dispatcher o lanzas corrutinas nuevas. Una forma de no olvidarlo es centralizarlo, por ejemplo en el `dbQuery` de los repositorios que viste en [Exposed DSL](../15-persistencia/04-exposed-dsl.md).

## Propagar a otros servicios

Si tu backend llama a otro servicio propio, manda el mismo ID. Así la historia sigue del otro lado:

```kotlin
HttpClient(OkHttp) {
  defaultRequest {
    MDC.get("callId")?.let { header(HttpHeaders.XRequestId, it) }
  }
}
```

Con proveedores externos, el header normalmente no sirve de nada del otro lado, pero **tus** logs de la llamada (petición, código de respuesta, duración) sí llevan el `callId`.

## Jobs: su propio ID

Un job programado no tiene una petición HTTP, así que nadie le asigna un ID. Sin uno, los logs de dos ejecuciones seguidas de la sincronización se mezclan. Genera uno por **ejecución**:

```kotlin
lanzarJobPeriodico("sincronizacion", 1.minutes) {
  val contexto = mapOf("job" to "sincronizacion", "ejecucionId" to UUID.randomUUID().toString())
  withContext(MDCContext(contexto)) {
    logger.info { "Ejecución iniciada" }
    sincronizar.ejecutar()
    logger.info { "Ejecución terminada" }
  }
}
```

Ahora buscar `ejecucionId = ...` muestra una ejecución completa; buscar `job = sincronizacion AND level = ERROR`, todas las fallas del job.

## Más campos de contexto

El mismo mecanismo sirve para cualquier dato que quieras en todos los logs de una petición:

```kotlin
MDC.put("empresaId", contexto.empresaId.value)
MDC.put("usuarioId", contexto.usuarioId.value)
```

Con `empresaId` en el MDC, "todos los errores de la empresa e-42 hoy" es una búsqueda simple. Usa **IDs**, no emails ni nombres ([Logging estructurado](01-logging-estructurado.md)).

## Más allá: trazas distribuidas

El correlation ID une logs. Una **traza distribuida** va más lejos: registra cada paso de la petición por varios servicios con su duración, y lo muestra como una línea de tiempo ("80 ms en la base, 600 ms esperando al proveedor").

El estándar es **OpenTelemetry**, y el header estándar para propagar el contexto entre servicios es `traceparent` (W3C Trace Context). Cuando un sistema crece a varios servicios, es el siguiente paso natural después del correlation ID.

-----

## Resumen en 5 líneas

1. Un correlation ID por petición (CallId: se reutiliza el `X-Request-Id` o se genera) une todos sus logs.
2. CallLogging lo pone en el MDC, y aparece en cada log sin pasarlo a mano; se devuelve en el header y en el error.
3. El MDC es thread-local: con corrutinas que cambian de hilo, usa `MDCContext` para que viaje con la corrutina.
4. Propágalo a tus otros servicios en el header, y genera un ID por ejecución en cada job.
5. Agrega al MDC IDs de contexto (empresa, usuario); el siguiente paso son las trazas distribuidas con OpenTelemetry.

-----

## Siguiente lección

[3. Métricas con Micrometer y Prometheus](03-metricas.md): números que avisan antes que los clientes.
