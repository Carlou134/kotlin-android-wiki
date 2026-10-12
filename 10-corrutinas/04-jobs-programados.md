# Jobs programados

## En una frase

Un job programado es una corrutina de larga vida que repite un trabajo cada cierto tiempo (sincronizar datos, purgar registros viejos, enviar reportes), atada al ciclo de vida del servidor para que arranque y se detenga con él.

-----

## Antes de empezar

Conviene que ya sepas:

* Scopes y dispatchers: [Scopes y dispatchers](02-scopes-y-dispatchers.md).
* Cancelación y supervisión: [Concurrencia estructurada](03-concurrencia-estructurada.md).
* Qué es un puerto y un use case: [Arquitectura hexagonal](../12-arquitectura/02-arquitectura-hexagonal.md).

Palabras nuevas (también están en el [Glosario](Glosario.md)):

* **Job programado:** Tarea que se ejecuta sola, cada cierto intervalo o a cierta hora.
* **Intervalo fijo entre ejecuciones (fixed delay):** Esperar X tiempo *después* de que termina cada ejecución.
* **Frecuencia fija (fixed rate):** Arrancar cada X tiempo, sin importar cuánto tardó la anterior.
* **Jitter:** Un pequeño retraso aleatorio que se suma para que muchas tareas no arranquen todas en el mismo instante.
* **Apagado ordenado (graceful shutdown):** Detener el servidor dejando que el trabajo en curso termine o se cancele de forma limpia.

-----

## Ejemplos típicos en un backend

* **Sincronización:** traer cada minuto el estado actualizado desde la API de un proveedor.
* **Purga:** borrar cada noche los registros más viejos que el período de retención.
* **Alertas por inactividad:** revisar cada 5 minutos qué dispositivos dejaron de reportar.
* **Reportes:** generar y enviar por email un resumen cada lunes.

## La forma más simple: un bucle con `delay`

```kotlin
fun CoroutineScope.lanzarPurga(purgar: PurgarRegistrosViejos): Job = launch {
  while (isActive) {
    purgar.ejecutar()
    delay(1.hours)
  }
}
```

* `while (isActive)` sale del bucle cuando cancelan el scope.
* `delay` es además un punto de suspensión: si cancelan mientras espera, sale en el acto.
* El job llama a un **use case** (puerto de entrada). La lógica de qué purgar vive en la capa application, no en el job. El job es solo un adapter primario más, como una ruta HTTP, pero disparado por el reloj.

## Un error no debe matar al job

Con el código de arriba, si `purgar.ejecutar()` lanza una excepción **una vez**, el bucle termina y el job no vuelve a correr hasta que reinicies el servidor. Nadie se entera.

```kotlin
fun CoroutineScope.lanzarJobPeriodico(
  nombre: String,
  intervalo: Duration,
  trabajo: suspend () -> Unit,
): Job = launch {
  while (isActive) {
    try {
      trabajo()
    } catch (e: CancellationException) {
      throw e
    } catch (e: Exception) {
      logger.error(e) { "El job $nombre falló; se reintenta en $intervalo" }
    }
    delay(intervalo)
  }
}
```

Cada ejecución está aislada: si una falla, se registra y la siguiente corre normalmente. Y la `CancellationException` se relanza, como viste en la lección anterior.

## Atarlo al ciclo de vida de Ktor

La `Application` de Ktor es un `CoroutineScope` que se cancela cuando el servidor se apaga. Lanzar los jobs ahí los ata al ciclo de vida sin código extra:

```kotlin
fun Application.configurarJobs() {
  val sincronizar by inject<SincronizarProveedores>()
  val purgar by inject<PurgarRegistrosViejos>()

  lanzarJobPeriodico("sincronizacion", 1.minutes) { sincronizar.ejecutar() }
  lanzarJobPeriodico("purga", 1.hours) { purgar.ejecutar() }
}
```

Esto vive en el **composition root** (`:app`), junto con las rutas: es el único lugar que decide qué se ejecuta y cuándo.

Si un job usa código bloqueante (JDBC), lánzalo en `Dispatchers.IO` o haz que el use case use `withContext(Dispatchers.IO)` internamente.

## Fixed delay vs fixed rate

Con `delay(intervalo)` *después* del trabajo, tienes **fixed delay**: si la sincronización tarda 40 segundos y el intervalo es 1 minuto, arranca cada 1 minuto y 40 segundos.

| | Fixed delay | Fixed rate |
| --- | --- | --- |
| Cuándo arranca la siguiente | X tiempo después de que termina la anterior | Cada X tiempo exacto |
| ¿Se pueden solapar? | Nunca | Sí, si una ejecución tarda más que X |
| Úsalo para | Casi todo: sincronizaciones, purgas | Cuando la hora exacta importa (y controlas el solapamiento) |

El bucle con `delay` es fixed delay por diseño, y eso es una ventaja: **una ejecución nunca se pisa con la siguiente**.

## El problema de las varias instancias

Todo lo anterior funciona con **un** servidor. Pero en producción sueles tener 2 o 3 instancias del mismo backend detrás de un balanceador. Cada una lanza sus propios jobs, así que la purga corre 3 veces y el reporte llega 3 veces por email.

Opciones:

| Estrategia | Cómo funciona | Contra |
| --- | --- | --- |
| Solo una instancia corre jobs | Una variable de entorno (`JOBS_ENABLED=true`) en una sola | Si esa instancia se cae, no corre nada |
| Lock en la base de datos | Antes de trabajar, cada instancia intenta tomar un lock o un *lease*; solo la que lo consigue trabaja | Más código, pero tolera caídas |
| Trabajo idempotente | El trabajo es seguro aunque corra dos veces | No siempre es posible (emails) |

El **lease** (un lock con fecha de vencimiento guardado en una tabla) es la solución más robusta, y se ve en la etapa de persistencia, junto con el bloqueo optimista. La idempotencia se ve en [Idempotencia](../17-resiliencia-e-integraciones/02-idempotencia.md).

## Jitter: que no arranquen todos juntos

Si 50 jobs arrancan a la vez cuando prende el servidor, todos golpean la base y las APIs externas en el mismo segundo. Un retraso inicial aleatorio los reparte:

```kotlin
launch {
  delay(Random.nextLong(0, 30_000).milliseconds) // entre 0 y 30 s
  while (isActive) { /* ... */ }
}
```

-----

## Resumen en 5 líneas

1. Un job programado es un bucle `while (isActive) { trabajo(); delay(intervalo) }` dentro de un scope con ciclo de vida.
2. Aísla cada ejecución con `try`/`catch` para que un error no mate al job, y relanza siempre `CancellationException`.
3. En Ktor, lánzalo en el scope de la `Application`, desde el composition root, y haz que llame a un use case.
4. El bucle con `delay` es fixed delay: nunca se solapan dos ejecuciones del mismo job.
5. Con varias instancias, cada una corre los jobs: usa un lease en la base, una sola instancia con jobs, o trabajo idempotente.

-----

## Siguiente lección

Terminaste las corrutinas. Vuelve al índice: [README principal](../README.md).
