# Concurrencia estructurada

## En una frase

La concurrencia estructurada organiza las corrutinas como un **árbol de padres e hijas**: un padre no termina hasta que terminan sus hijas, cancelar al padre cancela a las hijas, y un error de una hija sube al padre.

-----

## Antes de empezar

Conviene que ya sepas:

* Scopes, `launch` y `async`: [Scopes y dispatchers](02-scopes-y-dispatchers.md).
* `try`/`catch`/`finally`: [Excepciones](../07-null-safety-y-errores/02-excepciones.md).

Palabras nuevas (también están en el [Glosario](Glosario.md)):

* **Cancelación cooperativa:** La corrutina no se mata de golpe; se le avisa y ella se detiene en el próximo punto de suspensión.
* **`CancellationException`:** La excepción que se usa para cancelar una corrutina. No es un error: es la señal de "detente".
* **`SupervisorJob`:** Un `Job` cuyas hijas pueden fallar sin cancelar a sus hermanas.
* **Timeout:** Tiempo máximo que se le da a una operación antes de cancelarla.

-----

## El problema: tareas huérfanas

Sin estructura, este código tiene dos bugs:

```kotlin
suspend fun sincronizarTodo() {
  GlobalScope.launch { sincronizarProveedorA() }
  GlobalScope.launch { sincronizarProveedorB() }
  println("Listo") // ❌ mentira: las dos siguen corriendo
}
```

1. Dice "Listo" antes de terminar.
2. Si quien llamó se cancela (por ejemplo, el cliente cortó la petición), las sincronizaciones **siguen corriendo** sin dueño.

## Las tres reglas del árbol

```kotlin
suspend fun sincronizarTodo() = coroutineScope {   // padre
  launch { sincronizarProveedorA() }               // hija 1
  launch { sincronizarProveedorB() }               // hija 2
}   // no sale de acá hasta que terminen las dos
```

1. **El padre espera a sus hijas.** `coroutineScope` no devuelve hasta que todas terminan.
2. **Cancelar al padre cancela a las hijas.** Si cancelan `sincronizarTodo`, se cancelan A y B.
3. **Un error en una hija cancela al padre y a sus hermanas.** Si A lanza una excepción, B se cancela y el error sale de `coroutineScope`.

La regla 3 es correcta cuando las tareas son **partes de un mismo resultado**: si no puedes armar el resumen sin los pedidos, no tiene sentido seguir buscando las facturas.

## Cuando las tareas son independientes: supervisión

Pero a veces la regla 3 es justo lo que **no** quieres. Si sincronizas 3 proveedores y uno está caído, ¿por qué dejar sin sincronizar a los otros dos?

`supervisorScope` cambia la regla 3: el error de una hija **no** afecta a sus hermanas.

```kotlin
suspend fun sincronizarProveedores(proveedores: List<Proveedor>) = supervisorScope {
  proveedores.forEach { proveedor ->
    launch {
      try {
        sincronizar(proveedor)
      } catch (e: CancellationException) {
        throw e
      } catch (e: Exception) {
        logger.error(e) { "Falló la sincronización de ${proveedor.nombre}" }
      }
    }
  }
}
```

Cada hija maneja su propio error. Si una falla, las otras siguen.

`SupervisorJob()` es lo mismo pero para scopes de larga vida, como el de un servicio en segundo plano:

```kotlin
class SincronizadorService {
  private val scope = CoroutineScope(SupervisorJob() + Dispatchers.IO)

  fun detener() = scope.cancel()
}
```

| | `coroutineScope` / `Job` | `supervisorScope` / `SupervisorJob` |
| --- | --- | --- |
| Si una hija falla | Cancela a todas | Las demás siguen |
| Úsalo cuando | Las tareas forman un solo resultado | Las tareas son independientes |

**Trampa común:** `launch(SupervisorJob()) { ... }` **no** hace lo que parece. Le da a esa corrutina un padre nuevo y la saca del árbol, así que pierdes la regla 2. Usa `supervisorScope { }` o un scope creado con `SupervisorJob()`.

## La cancelación es cooperativa

Cancelar no mata la corrutina en el acto. Se le avisa, y ella se detiene en el **próximo punto de suspensión** (`delay`, una llamada `suspend`, etc.), que lanza `CancellationException`.

Un bucle de cálculo puro, sin puntos de suspensión, **nunca** se entera:

```kotlin
launch {
  for (registro in millonesDeRegistros) {
    procesar(registro) // CPU pura: no revisa la cancelación
  }
}
```

La solución es revisar la cancelación a mano:

```kotlin
launch {
  for (registro in millonesDeRegistros) {
    ensureActive() // lanza CancellationException si cancelaron
    procesar(registro)
  }
}
```

## La regla de oro: nunca te tragues `CancellationException`

`CancellationException` hereda de `Exception`. Entonces este código, que parece inocente, **rompe la cancelación**:

```kotlin
try {
  sincronizar(proveedor)
} catch (e: Exception) {           // ❌ atrapa también la cancelación
  logger.error(e) { "Falló" }
}
// la corrutina sigue como si nada, aunque la cancelaron
```

Lo correcto es relanzarla siempre:

```kotlin
try {
  sincronizar(proveedor)
} catch (e: CancellationException) {
  throw e                          // ✅ deja que la cancelación siga su curso
} catch (e: Exception) {
  logger.error(e) { "Falló" }
}
```

Lo mismo pasa con `runCatching { }`: atrapa **todo**, incluida la cancelación. Úsalo con cuidado dentro de corrutinas.

## Limpieza al cancelar

`finally` se ejecuta también cuando cancelan. Pero ojo: dentro de una corrutina cancelada, cualquier llamada `suspend` vuelve a lanzar `CancellationException`. Si la limpieza necesita suspender, envuélvela en `NonCancellable`:

```kotlin
try {
  procesarLote()
} finally {
  withContext(NonCancellable) {
    liberarLease() // suspend: sin NonCancellable no llegaría a ejecutarse
  }
}
```

## Timeouts

Una llamada a una API externa puede colgarse para siempre. Siempre ponle un límite:

```kotlin
val estado = withTimeout(5.seconds) {
  clienteProveedor.obtenerEstado(id)
} // lanza TimeoutCancellationException si tarda más
```

Si prefieres `null` en vez de una excepción:

```kotlin
val estado = withTimeoutOrNull(5.seconds) {
  clienteProveedor.obtenerEstado(id)
} ?: return EstadoOutcome.ProveedorNoResponde
```

`TimeoutCancellationException` es una `CancellationException`. Si la atrapas con un `catch (e: CancellationException)` genérico, la confundes con una cancelación real. Si quieres distinguirlas, atrápala antes de forma específica o usa `withTimeoutOrNull`.

## Errores en `async`

Con `async`, la excepción no se pierde: se guarda y se lanza **al llamar `await()`**. Y por la regla 3, dentro de un `coroutineScope` también cancela a las hermanas.

```kotlin
coroutineScope {
  val a = async { error("falló A") }
  val b = async { delay(1000); 42 }
  a.await() + b.await() // lanza "falló A" y b se cancela
}
```

-----

## Resumen en 5 líneas

1. Las corrutinas forman un árbol: el padre espera a sus hijas, cancelar al padre cancela a las hijas y un error de una hija cancela al grupo.
2. `supervisorScope` y `SupervisorJob` aíslan los errores cuando las tareas son independientes; no uses `launch(SupervisorJob())`.
3. La cancelación es cooperativa: ocurre en los puntos de suspensión o con `ensureActive()`.
4. Nunca te tragues `CancellationException`: relánzala; cuidado con `catch (e: Exception)` y `runCatching`.
5. Ponle timeout a todo lo externo con `withTimeout` o `withTimeoutOrNull`; para limpiar al cancelar, usa `NonCancellable`.

-----

## Siguiente lección

[4. Jobs programados](04-jobs-programados.md): tareas que se repiten solas mientras el servidor está prendido.
