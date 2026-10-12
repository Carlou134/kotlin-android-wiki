# Reintentos con backoff exponencial

## En una frase

Cuando una llamada a otro sistema falla por un problema **pasajero**, se reintenta esperando cada vez más (backoff exponencial) y con un poco de azar (jitter); cuando falla por un problema **permanente**, reintentar no sirve y solo empeora las cosas.

-----

## Antes de empezar

Conviene que ya sepas:

* Timeouts y por qué relanzar `CancellationException`: [Concurrencia estructurada](../10-corrutinas/03-concurrencia-estructurada.md).
* Funciones de orden superior: [Funciones de orden superior](../09-funcional/01-funciones-de-orden-superior.md).

Palabras nuevas (también están en el [Glosario](Glosario.md)):

* **Error transitorio:** Falla pasajera que probablemente se resuelva sola: un timeout, un servidor reiniciándose.
* **Error permanente:** Falla que se va a repetir igual: datos inválidos, credenciales incorrectas.
* **Backoff:** La espera entre un intento y el siguiente.
* **Backoff exponencial:** La espera se duplica en cada intento: 1 s, 2 s, 4 s, 8 s...
* **Jitter:** Azar que se suma a la espera para que muchos clientes no reintenten todos al mismo tiempo.
* **Tormenta de reintentos:** Cuando los reintentos de muchos clientes saturan a un servicio que ya estaba con problemas.

-----

## El problema: las redes fallan

Cuando tu backend llama a la API de un proveedor, pueden pasar muchas cosas que **no** tienen que ver con tu código:

* La conexión se corta a mitad de camino.
* El proveedor está desplegando y responde `503` durante 20 segundos.
* Un balanceador intermedio responde `502`.
* El proveedor te limita y responde `429 Too Many Requests`.

Muchas de esas fallas duran segundos. Si al primer error marcas la sincronización como fallida, pierdes datos por un problema que se habría resuelto solo.

## ¿Qué se reintenta y qué no?

La pregunta clave: **¿si lo intento de nuevo, igual, puede funcionar?**

| Situación | ¿Reintentar? | Por qué |
| --- | --- | --- |
| Timeout, conexión rechazada o cortada | ✅ | Problema de red, probablemente pasajero |
| `429 Too Many Requests` | ✅, respetando `Retry-After` | El proveedor te dice cuándo volver |
| `502`, `503`, `504` | ✅ | El servidor (o algo en el medio) no está disponible por ahora |
| `500 Internal Server Error` | ⚠️ Con cuidado | Puede ser pasajero o un bug del proveedor que fallará siempre |
| `400 Bad Request`, `422` | ❌ | Tus datos están mal: fallará igual |
| `401 Unauthorized` | ❌ (o renovar el token **una** vez) | Credenciales inválidas o vencidas |
| `403 Forbidden`, `404 Not Found` | ❌ | No tienes permiso o no existe: no va a cambiar |
| `409 Conflict` | ❌ | Es un conflicto de estado: hay que decidir qué hacer, no repetir |

**Condición previa:** solo reintentas operaciones **idempotentes** o protegidas con una clave de idempotencia. Si un `POST` que crea un pago dio timeout, puede que el pago **sí** se haya creado; reintentar a ciegas puede cobrar dos veces. Eso es la [lección 2](02-idempotencia.md).

## Por qué esperar cada vez más

Reintentar **inmediatamente** es peor que no reintentar: si el servidor está saturado, le mandas más carga justo cuando menos puede.

Con **backoff exponencial**, cada espera es el doble que la anterior:

```
intento 1: falla -> espera 1 s
intento 2: falla -> espera 2 s
intento 3: falla -> espera 4 s
intento 4: falla -> espera 8 s
```

Le das al sistema cada vez más tiempo para recuperarse. Y siempre con un **tope**: no tiene sentido esperar 17 minutos entre intentos.

## Por qué el azar (jitter)

Imaginá 1.000 instancias que fallaron en el mismo segundo porque el proveedor se cayó. Con backoff exponencial puro, **todas** reintentan exactamente al segundo 1, al 2, al 4... Cada reintento es un pico sincronizado que vuelve a tumbar al proveedor.

El jitter las desordena. La variante más simple y efectiva (*full jitter*) es esperar un tiempo **aleatorio** entre cero y el backoff calculado:

```
espera = random(0, min(tope, base × 2^intento))
```

## Una implementación

```kotlin
suspend fun <T> conReintentos(
  maxIntentos: Int = 4,
  base: Duration = 500.milliseconds,
  tope: Duration = 10.seconds,
  esReintentable: (Throwable) -> Boolean,
  bloque: suspend () -> T,
): T {
  var intento = 0
  while (true) {
    try {
      return bloque()
    } catch (e: CancellationException) {
      throw e                                    // nunca reintentes una cancelación
    } catch (e: Exception) {
      intento++
      if (intento >= maxIntentos || !esReintentable(e)) throw e
      val maximo = minOf(tope, base * (1 shl intento))
      delay(Random.nextLong(0, maximo.inWholeMilliseconds + 1).milliseconds)
    }
  }
}
```

Y quien la usa decide qué se reintenta:

```kotlin
val estado = conReintentos(esReintentable = ::esErrorTransitorio) {
  clienteProveedor.obtenerEstado(dispositivoId)
}
```

Fíjate en el `catch (e: CancellationException) { throw e }`: lo viste en [Concurrencia estructurada](../10-corrutinas/03-concurrencia-estructurada.md). Un reintento que se traga la cancelación mantiene vivo un job que pidieron detener.

## En Ktor Client: `HttpRequestRetry`

Ktor Client trae un plugin que hace todo esto:

```kotlin
val cliente = HttpClient(OkHttp) {
  install(HttpRequestRetry) {
    retryOnServerErrors(maxRetries = 3)              // reintenta respuestas 5xx
    retryOnException(maxRetries = 3, retryOnTimeout = true)
    exponentialDelay()                               // backoff exponencial con jitter
  }
}
```

Para reglas propias (por ejemplo, reintentar `429` y no `500`), el plugin permite definir `retryIf { request, response -> ... }`.

## Tres trampas

**1. Reintentos en capas que se multiplican.** El cliente HTTP reintenta 3 veces, el use case envuelve todo en otros 3 reintentos y el job reintenta la ejecución 3 veces: una falla se convierte en **27** llamadas. Decide **una** capa responsable de reintentar.

**2. Sin límite de tiempo total.** 4 intentos con timeout de 30 segundos pueden tardar más de 2 minutos. Además del número de intentos, define un tiempo máximo total con `withTimeout` alrededor de todo.

**3. Seguir golpeando a un servicio caído.** Si el proveedor lleva 10 minutos caído, cada ejecución gasta reintentos inútiles. El patrón **circuit breaker** lo resuelve: después de N fallas seguidas, deja de llamar durante un rato y falla rápido. Para jobs periódicos, muchas veces alcanza con que la **siguiente ejecución** del job sea el reintento.

-----

## Resumen en 5 líneas

1. Reintenta solo errores transitorios (timeouts, `429`, `502`, `503`, `504`); los permanentes (`400`, `401`, `403`, `404`) fallarán igual.
2. Solo reintenta operaciones idempotentes o protegidas con una clave de idempotencia.
3. Backoff exponencial con tope le da tiempo al sistema; el jitter evita que todos reintenten a la vez.
4. Nunca reintentes una `CancellationException`; en Ktor Client, el plugin `HttpRequestRetry` resuelve lo básico.
5. Una sola capa reintenta, con un tiempo máximo total; ante caídas largas, un circuit breaker o la próxima ejecución del job.

-----

## Siguiente lección

[2. Idempotencia](02-idempotencia.md): cómo reintentar sin duplicar.
