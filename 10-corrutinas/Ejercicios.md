# Ejercicios de corrutinas

Cuaderno de práctica de toda la carpeta: `suspend`, `launch` y `async`, scopes y dispatchers, concurrencia estructurada, cancelación, timeouts y jobs periódicos. Cada solución es un programa completo con `fun main()`: puedes pegarlo en un archivo `.kt` y ejecutarlo.

Requisitos previos: [`suspend`](01-suspend.md), [Scopes y dispatchers](02-scopes-y-dispatchers.md), [Concurrencia estructurada](03-concurrencia-estructurada.md) y [Jobs programados](04-jobs-programados.md).

**Cómo usarlo:** intenta cada ejercicio **antes** de abrir la solución. Si te trabas, vuelve a la lección, no a la solución. Mirar la respuesta antes de pelearla es leer, no entrenar.

**Dependencia:** las corrutinas no vienen en la librería estándar. Agrega esto a tu `build.gradle.kts`:

```kotlin
dependencies {
  implementation("org.jetbrains.kotlinx:kotlinx-coroutines-core:1.9.0")
}
```

**Sobre los tiempos:** varios ejercicios miden cuánto tarda algo. Los números redondeados que ves en las salidas pueden variar un poco en tu máquina. Lo que importa es la **proporción** (1 s contra 3 s), no el milisegundo exacto.

-----

## Ejercicios guiados

### Ejercicio 1: bloquear contra suspender

**Objetivo:** ver con números la diferencia entre `Thread.sleep` y `delay`.

**Contexto:** la lección dice que `delay` libera el hilo y `Thread.sleep` no. Vamos a comprobarlo con tres tareas en `runBlocking`, que usa **un solo hilo**.

**Instrucciones:**

1. Dentro de un `coroutineScope`, lanza 3 corrutinas con `launch`. Cada una espera 1 segundo con `Thread.sleep(1000)` e imprime su número.
2. Mide cuánto tarda el bloque con `measureTime`.
3. Repite lo mismo con `delay(1000)`.
4. Antes de ejecutar, predice los dos tiempos.

<details>
<summary>Solución</summary>

```kotlin
import kotlinx.coroutines.*
import kotlin.time.measureTime

fun main() = runBlocking {
  val conSleep = measureTime {
    coroutineScope {
      repeat(3) { i ->
        launch {
          Thread.sleep(1000) // bloquea el único hilo
          println("sleep $i")
        }
      }
    }
  }
  println("Con Thread.sleep: ~${conSleep.inWholeSeconds} s")

  val conDelay = measureTime {
    coroutineScope {
      repeat(3) { i ->
        launch {
          delay(1000) // pausa la corrutina, el hilo queda libre
          println("delay $i")
        }
      }
    }
  }
  println("Con delay: ~${conDelay.inWholeSeconds} s")
}
```

Salida:

```
sleep 0
sleep 1
sleep 2
Con Thread.sleep: ~3 s
delay 0
delay 1
delay 2
Con delay: ~1 s
```

</details>

**Qué observar:** el código de los dos bloques es **idéntico** salvo una línea. Con `Thread.sleep`, la primera corrutina toma el único hilo y las otras dos esperan su turno: 3 segundos. Con `delay`, las tres se pausan a la vez y el hilo queda libre: 1 segundo. Esto es lo que dice la idea equivocada de la lección 1: `launch` no hace magia si adentro llamas a algo que bloquea.

-----

### Ejercicio 2: predice el orden

**Objetivo:** entender **cuándo** empieza a correr una corrutina lanzada con `launch`.

**Contexto:** mucha gente cree que `launch` ejecuta el bloque en el acto. No es así.

**Instrucciones:** sin ejecutarlo, escribe en un papel el orden en que se imprimen los números. Después compruébalo.

```kotlin
import kotlinx.coroutines.*

fun main() = runBlocking {
  println("1")
  launch {
    println("2")
    delay(300)
    println("3")
  }
  launch {
    delay(100)
    println("4")
  }
  println("5")
  delay(200)
  println("6")
}
```

<details>
<summary>Solución</summary>

Salida:

```
1
5
2
4
6
3
```

Paso a paso, en el único hilo de `runBlocking`:

| Tiempo | Qué pasa |
| --- | --- |
| 0 ms | Imprime `1`. Los dos `launch` solo **programan** las corrutinas. Imprime `5`. |
| 0 ms | `main` llega a `delay(200)` y se pausa. Recién ahora el hilo queda libre: la primera corrutina imprime `2` y se pausa; la segunda se pausa. |
| 100 ms | La segunda corrutina imprime `4`. |
| 200 ms | `main` retoma e imprime `6`. |
| 300 ms | La primera corrutina imprime `3`. `runBlocking` esperaba a sus hijas, así que el programa termina recién ahora. |

</details>

**Qué observar:** el `2` no sale justo después del `1`. `launch` deja la corrutina en la cola, y esta solo corre cuando el hilo se libera, o sea, en el próximo punto de suspensión de quien la lanzó. Y fíjate en el `3`: sale **después** del último `println` de `main`, porque `runBlocking` es un padre que espera a sus hijas (regla 1 de la lección 3).

-----

### Ejercicio 3: en paralelo con `async`

**Objetivo:** usar `async` para hacer dos consultas independientes a la vez.

**Contexto:** el resumen de un cliente necesita sus pedidos y sus facturas. Cada consulta tarda 300 ms y no depende de la otra.

**Instrucciones:**

1. Escribe `resumenSecuencial`, que llama a las dos funciones una detrás de la otra.
2. Escribe `resumenParalelo`, que las lanza con `async` dentro de un `coroutineScope`.
3. Mide las dos versiones.

```kotlin
data class Resumen(val pedidos: List<String>, val facturas: List<String>)

suspend fun buscarPedidos(clienteId: String): List<String> {
  delay(300)
  return listOf("p1", "p2")
}

suspend fun buscarFacturas(clienteId: String): List<String> {
  delay(300)
  return listOf("f1")
}
```

<details>
<summary>Solución</summary>

```kotlin
import kotlinx.coroutines.*
import kotlin.time.measureTime

data class Resumen(val pedidos: List<String>, val facturas: List<String>)

suspend fun buscarPedidos(clienteId: String): List<String> {
  delay(300)
  return listOf("p1", "p2")
}

suspend fun buscarFacturas(clienteId: String): List<String> {
  delay(300)
  return listOf("f1")
}

suspend fun resumenSecuencial(clienteId: String) =
  Resumen(buscarPedidos(clienteId), buscarFacturas(clienteId))

suspend fun resumenParalelo(clienteId: String) = coroutineScope {
  val pedidos = async { buscarPedidos(clienteId) }
  val facturas = async { buscarFacturas(clienteId) }
  Resumen(pedidos.await(), facturas.await())
}

fun main() = runBlocking {
  val t1 = measureTime { println(resumenSecuencial("c1")) }
  println("Secuencial: ~${t1.inWholeMilliseconds / 100 * 100} ms")

  val t2 = measureTime { println(resumenParalelo("c1")) }
  println("Paralelo: ~${t2.inWholeMilliseconds / 100 * 100} ms")
}
```

Salida:

```
Resumen(pedidos=[p1, p2], facturas=[f1])
Secuencial: ~600 ms
Resumen(pedidos=[p1, p2], facturas=[f1])
Paralelo: ~300 ms
```

</details>

**Qué observar:** la trampa clásica es escribir `val pedidos = async { buscarPedidos(id) }.await()`. Eso **espera en el acto** y la segunda consulta no arranca hasta que termina la primera: vuelves a tener 600 ms con más código. La regla es lanzar **todos** los `async` primero y hacer los `await()` después.

-----

### Ejercicio 4: ¿en qué hilo estoy?

**Objetivo:** ver que `withContext` cambia de hilo y **vuelve** al hilo original.

**Contexto:** tienes una consulta JDBC (bloqueante) y un cálculo pesado. Cada uno debería correr en su dispatcher.

**Instrucciones:**

1. Escribe `consultarBase()`, que corre en `Dispatchers.IO`, imprime en qué hilo está y simula JDBC con `Thread.sleep(100)`.
2. Escribe `calcularTotal()`, que corre en `Dispatchers.Default` e imprime en qué hilo está.
3. En `main`, imprime el hilo antes, entre y después de cada llamada.

Usa esta función para que los nombres de hilo se lean fácil:

```kotlin
fun hilo() = Thread.currentThread().name.substringBefore("-worker")
```

<details>
<summary>Solución</summary>

```kotlin
import kotlinx.coroutines.*

fun hilo() = Thread.currentThread().name.substringBefore("-worker")

suspend fun consultarBase(): String = withContext(Dispatchers.IO) {
  println("consulta en: ${hilo()}")
  Thread.sleep(100) // simula un driver JDBC: bloquea, pero un hilo del pool de IO
  "pedido p1"
}

suspend fun calcularTotal(): Int = withContext(Dispatchers.Default) {
  println("cálculo en: ${hilo()}")
  (1..1000).sum()
}

fun main() = runBlocking {
  println("empieza en: ${hilo()}")
  val pedido = consultarBase()
  println("vuelve a: ${hilo()} con $pedido")
  val total = calcularTotal()
  println("vuelve a: ${hilo()} con $total")
}
```

Salida:

```
empieza en: main
consulta en: DefaultDispatcher
vuelve a: main con pedido p1
cálculo en: DefaultDispatcher
vuelve a: main con 500500
```

</details>

**Qué observar:** `withContext` es como una llamada normal: va a otro hilo, hace el trabajo y **te devuelve el resultado en el hilo donde estabas**. No hay callbacks. Y mira el nombre: `IO` y `Default` muestran el mismo prefijo porque comparten el mismo grupo de hilos por dentro. Lo que cambia es **cuántos** pueden usar a la vez (núcleos de CPU para `Default`, 64 para `IO`).

-----

### Ejercicio 5: el "Listo" mentiroso

**Objetivo:** arreglar tareas huérfanas con concurrencia estructurada.

**Contexto:** este es el código del principio de la lección 3. Ejecútalo y mira qué imprime.

```kotlin
import kotlinx.coroutines.*

suspend fun sincronizarProveedor(nombre: String, ms: Long) {
  delay(ms)
  println("$nombre sincronizado")
}

@OptIn(DelicateCoroutinesApi::class)
suspend fun sincronizarTodo() {
  GlobalScope.launch { sincronizarProveedor("A", 200) }
  GlobalScope.launch { sincronizarProveedor("B", 300) }
  println("Listo")
}

fun main() = runBlocking {
  sincronizarTodo()
  println("main terminó")
}
```

**Instrucciones:**

1. Predice la salida y compruébala.
2. Reescribe `sincronizarTodo` para que `Listo` se imprima **solo** cuando las dos sincronizaciones terminaron.

<details>
<summary>Solución</summary>

Salida del código original:

```
Listo
main terminó
```

Las sincronizaciones **nunca** se imprimen. `runBlocking` no espera a las corrutinas de `GlobalScope` porque no son sus hijas. `main` termina, la JVM se apaga y se las lleva puestas.

Versión corregida:

```kotlin
import kotlinx.coroutines.*

suspend fun sincronizarProveedor(nombre: String, ms: Long) {
  delay(ms)
  println("$nombre sincronizado")
}

suspend fun sincronizarTodo() {
  coroutineScope {
    launch { sincronizarProveedor("A", 200) }
    launch { sincronizarProveedor("B", 300) }
  }
  println("Listo") // afuera del bloque: coroutineScope ya esperó a las dos
}

fun main() = runBlocking {
  sincronizarTodo()
  println("main terminó")
}
```

Salida:

```
A sincronizado
B sincronizado
Listo
main terminó
```

</details>

**Qué observar:** fíjate **dónde** quedó el `println("Listo")`. Si lo dejas como última línea **dentro** del `coroutineScope { }`, vuelve a mentir: el cuerpo del bloque termina enseguida, y la espera a las hijas ocurre **al cerrar la llave**. Por eso la línea va después. Y el `@OptIn` del original no está de adorno: la librería marca `GlobalScope` como API delicada justamente por esto.

-----

### Ejercicio 6: un proveedor caído

**Objetivo:** elegir entre `coroutineScope` y `supervisorScope` según si las tareas son independientes.

**Contexto:** sincronizas tres proveedores. El B siempre falla.

```kotlin
suspend fun sincronizar(nombre: String, ms: Long) {
  delay(ms)
  if (nombre == "B") error("$nombre está caído")
  println("$nombre ok")
}

suspend fun sincronizarTodos() = coroutineScope {
  launch { sincronizar("A", 100) }
  launch { sincronizar("B", 50) }
  launch { sincronizar("C", 200) }
}
```

**Instrucciones:**

1. Llama a `sincronizarTodos()` desde `main` dentro de un `try`/`catch` que imprima el mensaje del error. Predice qué se imprime.
2. Reescribe la función para que A y C terminen aunque B falle, y que el error de B quede registrado con un `println`.

<details>
<summary>Solución</summary>

```kotlin
import kotlinx.coroutines.*

suspend fun sincronizar(nombre: String, ms: Long) {
  delay(ms)
  if (nombre == "B") error("$nombre está caído")
  println("$nombre ok")
}

suspend fun sincronizarTodos() = coroutineScope {
  launch { sincronizar("A", 100) }
  launch { sincronizar("B", 50) }
  launch { sincronizar("C", 200) }
}

suspend fun sincronizarIndependientes() = supervisorScope {
  listOf("A" to 100L, "B" to 50L, "C" to 200L).forEach { (nombre, ms) ->
    launch {
      try {
        sincronizar(nombre, ms)
      } catch (e: CancellationException) {
        throw e
      } catch (e: Exception) {
        println("$nombre falló: ${e.message}")
      }
    }
  }
}

fun main() = runBlocking {
  println("--- coroutineScope ---")
  try {
    sincronizarTodos()
  } catch (e: IllegalStateException) {
    println("Error: ${e.message}")
  }

  println("--- supervisorScope ---")
  sincronizarIndependientes()
}
```

Salida:

```
--- coroutineScope ---
Error: B está caído
--- supervisorScope ---
B falló: B está caído
A ok
C ok
```

</details>

**Qué observar:** con `coroutineScope`, B falla a los 50 ms y **cancela** a A y a C antes de que terminen: no se imprime ningún `ok`. Eso es correcto si los tres forman un solo resultado, y un desastre si son independientes. Y ojo: con `supervisorScope` el `try`/`catch` dentro de cada `launch` **no es opcional**. Si lo sacas, A y C siguen terminando, pero el error de B no tiene quién lo maneje y termina impreso como un stack trace en la consola, en vez de un registro tuyo.

-----

### Ejercicio 7: el bucle que no se entera

**Objetivo:** ver que la cancelación es cooperativa y arreglarlo con `ensureActive()`.

**Contexto:** un proceso pesado recorre registros. Lo cancelas a los 250 ms.

```kotlin
import kotlinx.coroutines.*

fun main() = runBlocking {
  val job = launch(Dispatchers.Default) {
    for (i in 1..5) {
      Thread.sleep(100) // simula cálculo pesado: no es un punto de suspensión
      println("registro $i procesado")
    }
  }
  delay(250)
  println("cancelando...")
  job.cancelAndJoin()
  println("fin")
}
```

**Instrucciones:**

1. Predice la salida y compruébala.
2. Agrega **una línea** para que el bucle se detenga en cuanto lo cancelen.

<details>
<summary>Solución</summary>

Salida del código original:

```
registro 1 procesado
registro 2 procesado
cancelando...
registro 3 procesado
registro 4 procesado
registro 5 procesado
fin
```

La cancelación llega a los 250 ms, pero el bucle no tiene ningún punto de suspensión donde enterarse. Procesa los 5 registros y `cancelAndJoin()` espera a que termine.

Versión corregida:

```kotlin
import kotlinx.coroutines.*

fun main() = runBlocking {
  val job = launch(Dispatchers.Default) {
    for (i in 1..5) {
      ensureActive() // lanza CancellationException si cancelaron
      Thread.sleep(100)
      println("registro $i procesado")
    }
  }
  delay(250)
  println("cancelando...")
  job.cancelAndJoin()
  println("fin")
}
```

Salida:

```
registro 1 procesado
registro 2 procesado
cancelando...
registro 3 procesado
fin
```

</details>

**Qué observar:** el registro 3 se procesa igual. Cuando `ensureActive()` revisó, a los 200 ms, todavía no habían cancelado. Para en la **siguiente** revisión. La cancelación cooperativa nunca corta a mitad de una operación: corta entre una y otra. Por eso el job corre en `Dispatchers.Default`: si corriera en el hilo de `runBlocking`, el `Thread.sleep` no dejaría ni siquiera que `main` llegue a cancelarlo.

-----

### Ejercicio 8: el `catch` que se traga la cancelación

**Objetivo:** detectar el bug de `catch (e: Exception)` dentro de una corrutina.

**Contexto:** alguien quería que un error en una vuelta no cortara el bucle. Parece razonable.

```kotlin
import kotlinx.coroutines.*

fun main() = runBlocking {
  val job = launch {
    repeat(5) { vuelta ->
      try {
        delay(100)
        println("vuelta $vuelta")
      } catch (e: Exception) {
        println("error: ${e::class.simpleName}")
      }
    }
    println("el bucle terminó")
  }
  delay(250)
  job.cancelAndJoin()
  println("fin")
}
```

**Instrucciones:**

1. Predice la salida. ¿Cuántas veces aparece `error`?
2. Corrige el `catch` para que la cancelación siga su curso.

<details>
<summary>Solución</summary>

Salida del código original:

```
vuelta 0
vuelta 1
error: JobCancellationException
error: JobCancellationException
error: JobCancellationException
el bucle terminó
fin
```

Al cancelar, el `delay` de la vuelta 2 lanza `CancellationException`, el `catch` la atrapa y el bucle sigue. En las vueltas 3 y 4, `delay` vuelve a lanzarla al instante porque la corrutina ya está cancelada. Tres "errores" que no son errores, y un bucle que terminó como si nada.

Versión corregida:

```kotlin
import kotlinx.coroutines.*

fun main() = runBlocking {
  val job = launch {
    repeat(5) { vuelta ->
      try {
        delay(100)
        println("vuelta $vuelta")
      } catch (e: CancellationException) {
        throw e
      } catch (e: Exception) {
        println("error: ${e::class.simpleName}")
      }
    }
    println("el bucle terminó")
  }
  delay(250)
  job.cancelAndJoin()
  println("fin")
}
```

Salida:

```
vuelta 0
vuelta 1
fin
```

</details>

**Qué observar:** aquí hubo suerte: como cada `delay` vuelve a lanzar la excepción, el bucle termina igual. Si en vez de `repeat(5)` fuera un `while (true)`, el bucle **nunca** saldría y quedaría girando, imprimiendo errores sin parar. Y como `cancelAndJoin()` espera a que termine, el programa quedaría colgado. El orden de los `catch` importa: el específico (`CancellationException`) va **antes** que el general.

-----

### Ejercicio 9: proveedores con timeout

**Objetivo:** combinar `async` con `withTimeoutOrNull` para que un proveedor lento no frene a los demás.

**Contexto:** consultas el estado de tres proveedores en paralelo. Uno tarda 3 segundos. No quieres esperar más de 500 ms por ninguno.

```kotlin
suspend fun consultarProveedor(nombre: String, ms: Long): String {
  delay(ms)
  return "$nombre: activo"
}

val proveedores = listOf("rapido" to 100L, "lento" to 3000L, "normal" to 300L)
```

**Instrucciones:**

1. Lanza las tres consultas en paralelo con `async`.
2. Cada consulta tiene un máximo de 500 ms. Si no responde, el resultado es `"<nombre>: no responde"`.
3. Imprime los tres estados, en el orden de la lista, y cuánto tardó todo.

<details>
<summary>Solución</summary>

```kotlin
import kotlinx.coroutines.*
import kotlin.time.Duration.Companion.milliseconds
import kotlin.time.measureTime

suspend fun consultarProveedor(nombre: String, ms: Long): String {
  delay(ms)
  return "$nombre: activo"
}

val proveedores = listOf("rapido" to 100L, "lento" to 3000L, "normal" to 300L)

fun main() = runBlocking {
  val tiempo = measureTime {
    val estados = proveedores
      .map { (nombre, ms) ->
        async {
          withTimeoutOrNull(500.milliseconds) { consultarProveedor(nombre, ms) }
            ?: "$nombre: no responde"
        }
      }
      .map { it.await() }

    estados.forEach(::println)
  }
  println("Tardó ~${tiempo.inWholeMilliseconds / 100 * 100} ms")
}
```

Salida:

```
rapido: activo
lento: no responde
normal: activo
Tardó ~500 ms
```

</details>

**Qué observar:** sin timeout, el total sería 3 segundos: lo que tarda el **más lento**. Con timeout, el peor caso queda acotado a 500 ms. Y los resultados salen en el orden de la lista, no en el orden en que terminaron, porque el segundo `map` hace los `await()` en orden. Usamos `withTimeoutOrNull` y no `withTimeout` porque aquí "no responde" es un resultado esperado, no un error que deba subir.

-----

### Ejercicio 10: la limpieza que no llega

**Objetivo:** usar `NonCancellable` cuando la limpieza necesita suspender.

**Contexto:** un proceso toma un lease y lo libera en el `finally`. Liberar el lease es una llamada `suspend` a la base.

```kotlin
import kotlinx.coroutines.*

suspend fun liberarLease() {
  delay(50) // simula la llamada a la base
  println("lease liberado")
}

fun main() = runBlocking {
  val job = launch {
    try {
      println("procesando lote...")
      delay(1000)
    } finally {
      println("limpiando")
      liberarLease()
    }
  }
  delay(100)
  job.cancelAndJoin()
  println("fin")
}
```

**Instrucciones:**

1. Predice la salida. ¿Aparece `lease liberado`?
2. Corrige el `finally` para que el lease siempre se libere.

<details>
<summary>Solución</summary>

Salida del código original:

```
procesando lote...
limpiando
fin
```

El `finally` **sí** se ejecuta: por eso ves `limpiando`. Pero la corrutina ya está cancelada, así que el `delay` dentro de `liberarLease()` lanza `CancellationException` en el acto y la línea `lease liberado` nunca llega.

Versión corregida:

```kotlin
import kotlinx.coroutines.*

suspend fun liberarLease() {
  delay(50)
  println("lease liberado")
}

fun main() = runBlocking {
  val job = launch {
    try {
      println("procesando lote...")
      delay(1000)
    } finally {
      println("limpiando")
      withContext(NonCancellable) {
        liberarLease()
      }
    }
  }
  delay(100)
  job.cancelAndJoin()
  println("fin")
}
```

Salida:

```
procesando lote...
limpiando
lease liberado
fin
```

</details>

**Qué observar:** el bug es silencioso: no hay stack trace ni error, solo una línea que falta. En producción, ese lease queda tomado hasta que venza, y ninguna otra instancia puede trabajar mientras tanto. Usa `NonCancellable` **solo** en limpiezas cortas: un bloque que no se puede cancelar y que se cuelga, cuelga el apagado del servidor.

-----

### Ejercicio 11: un job que sobrevive a sus errores

**Objetivo:** escribir un job periódico que no muere con el primer error.

**Contexto:** un job sincroniza cada 100 ms. La segunda ejecución falla porque el proveedor está caído.

```kotlin
import kotlinx.coroutines.*
import kotlin.time.Duration
import kotlin.time.Duration.Companion.milliseconds

fun CoroutineScope.lanzarJobPeriodico(
  nombre: String,
  intervalo: Duration,
  trabajo: suspend () -> Unit,
): Job = launch {
  while (isActive) {
    trabajo()
    delay(intervalo)
  }
}

fun main() = runBlocking {
  var ejecucion = 0
  val job = lanzarJobPeriodico("sync", 100.milliseconds) {
    ejecucion++
    if (ejecucion == 2) error("proveedor caído")
    println("[$ejecucion] sincronización ok")
  }
  delay(350)
  job.cancelAndJoin()
  println("servidor apagado")
}
```

**Instrucciones:**

1. Ejecútalo y mira qué pasa con el error.
2. Corrige `lanzarJobPeriodico` para que cada ejecución esté aislada: si una falla, se registra con `println` y la siguiente corre normalmente.

<details>
<summary>Solución</summary>

Salida del código original:

```
[1] sincronización ok
Exception in thread "main" java.lang.IllegalStateException: proveedor caído
	...
```

La excepción sale del bucle, mata al job y, como `runBlocking` no es un supervisor, tumba **todo el programa**. Ni siquiera llega a imprimir `servidor apagado`.

Versión corregida:

```kotlin
import kotlinx.coroutines.*
import kotlin.time.Duration
import kotlin.time.Duration.Companion.milliseconds

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
      println("[$nombre] falló: ${e.message}; se reintenta en $intervalo")
    }
    delay(intervalo)
  }
}

fun main() = runBlocking {
  var ejecucion = 0
  val job = lanzarJobPeriodico("sync", 100.milliseconds) {
    ejecucion++
    if (ejecucion == 2) error("proveedor caído")
    println("[$ejecucion] sincronización ok")
  }
  delay(350)
  job.cancelAndJoin()
  println("servidor apagado")
}
```

Salida:

```
[1] sincronización ok
[sync] falló: proveedor caído; se reintenta en 100ms
[3] sincronización ok
[4] sincronización ok
servidor apagado
```

</details>

**Qué observar:** aquí el error al menos se vio, en rojo y tumbando el programa. En un servidor el síntoma es peor: como dice la lección 4, el job deja de correr y nadie se entera hasta que alguien nota que los datos están viejos. Fíjate también en que `trabajo` es una lambda `suspend () -> Unit`: el job no sabe **qué** hace, solo **cuándo**. Es la misma separación que la lección pide entre el job y el use case.

-----

## Retos

### Reto 1: máximo 3 a la vez

**Misión:** tienes un cliente HTTP **bloqueante** de un proveedor que solo acepta 3 conexiones simultáneas. Consulta 9 pedidos de forma que nunca haya más de 3 llamadas a la vez. Mide el tiempo con el límite y sin él (con `Dispatchers.IO` a secas).

```kotlin
fun clienteBloqueante(id: Int): String {
  Thread.sleep(300) // bloquea, como un cliente HTTP viejo
  return "pedido $id"
}
```

**Pista:** mira la sección "Limitar la concurrencia" de la lección 2. Cada consulta va en su propio `async`, y dentro de cada uno, un `withContext` con el dispatcher limitado.

<details>
<summary>Solución</summary>

```kotlin
import kotlinx.coroutines.*
import kotlin.time.measureTime

fun clienteBloqueante(id: Int): String {
  Thread.sleep(300)
  return "pedido $id"
}

val apiProveedor = Dispatchers.IO.limitedParallelism(3)

suspend fun consultarTodos(ids: List<Int>, dispatcher: CoroutineDispatcher): List<String> =
  coroutineScope {
    ids.map { id -> async { withContext(dispatcher) { clienteBloqueante(id) } } }
      .map { it.await() }
  }

fun main() = runBlocking {
  val ids = (1..9).toList()

  val conLimite = measureTime { consultarTodos(ids, apiProveedor) }
  println("Con límite de 3: ~${conLimite.inWholeMilliseconds / 100 * 100} ms")

  val sinLimite = measureTime { consultarTodos(ids, Dispatchers.IO) }
  println("Sin límite: ~${sinLimite.inWholeMilliseconds / 100 * 100} ms")
}
```

Salida:

```
Con límite de 3: ~900 ms
Sin límite: ~300 ms
```

Con el límite, las 9 llamadas van en 3 tandas de 3: 900 ms. Sin él, las 9 corren a la vez en el pool de IO: 300 ms, y el proveedor recibe 9 conexiones que no acepta.

Ahora la trampa: si cambias `Thread.sleep(300)` por `delay(300)` (un cliente que suspende, como el de Ktor), `limitedParallelism(3)` **deja de limitar**. Una corrutina suspendida no ocupa hilo, así que entran las 9. `limitedParallelism` limita **hilos**, no llamadas. Para limitar llamadas a algo que suspende se usa un `Semaphore` de `kotlinx.coroutines.sync` con `withPermit { }`.

</details>

-----

### Reto 2: reintentar con espera creciente

**Misión:** escribe `reintentar(intentos, esperaInicial, bloque)`, que ejecuta `bloque` y, si falla, espera y vuelve a intentar, duplicando la espera cada vez. Si el último intento falla, la excepción tiene que salir. Y por supuesto, la cancelación no se reintenta. Pruébala con:

```kotlin
var llamadas = 0

suspend fun consultarInestable(): String {
  llamadas++
  if (llamadas < 3) error("timeout del proveedor")
  return "estado: activo"
}
```

**Pista:** un `repeat(intentos - 1)` con un `try` adentro, y el último intento afuera del `repeat`, sin `try`. Para salir del `repeat` cuando sale bien, `return bloque()` funciona porque `repeat` es `inline`: es el `return` que se escapa del [ejercicio 9 de programación funcional](../09-funcional/Ejercicios.md#ejercicio-9-return-que-se-escapa), esta vez a propósito.

<details>
<summary>Solución</summary>

```kotlin
import kotlinx.coroutines.*
import kotlin.time.Duration
import kotlin.time.Duration.Companion.milliseconds

suspend fun <T> reintentar(
  intentos: Int,
  esperaInicial: Duration,
  bloque: suspend () -> T,
): T {
  var espera = esperaInicial
  repeat(intentos - 1) { intento ->
    try {
      return bloque()
    } catch (e: CancellationException) {
      throw e
    } catch (e: Exception) {
      println("intento ${intento + 1} falló: ${e.message}; espero $espera")
      delay(espera)
      espera *= 2
    }
  }
  return bloque() // último intento: si falla, la excepción sale
}

var llamadas = 0

suspend fun consultarInestable(): String {
  llamadas++
  if (llamadas < 3) error("timeout del proveedor")
  return "estado: activo"
}

fun main() = runBlocking {
  println(reintentar(4, 100.milliseconds) { consultarInestable() })

  try {
    reintentar(2, 100.milliseconds) { error("proveedor caído") }
  } catch (e: IllegalStateException) {
    println("me rindo: ${e.message}")
  }
}
```

Salida:

```
intento 1 falló: timeout del proveedor; espero 100ms
intento 2 falló: timeout del proveedor; espero 200ms
estado: activo
intento 1 falló: proveedor caído; espero 100ms
me rindo: proveedor caído
```

Duplicar la espera (backoff exponencial) le da aire a un proveedor que está saturado: si todos los clientes reintentan cada 100 ms fijos, lo terminan de tumbar. En producción se le suma jitter, como en la lección 4, para que los reintentos de muchos clientes no caigan todos juntos. El tema completo de reintentos se ve en [Resiliencia e integraciones](../17-resiliencia-e-integraciones/README.md).

</details>

-----

### Reto 3: frecuencia fija sin solaparse

**Misión:** escribe `lanzarFrecuenciaFija(intervalo, trabajo)`, que intenta arrancar el trabajo cada `intervalo`, sin importar cuánto tarda (fixed rate). Si la ejecución anterior **todavía está corriendo**, ese tick se omite. Pruébalo con un intervalo de 100 ms, un trabajo que tarda 250 ms, y cancela a los 550 ms.

**Pista:** el bucle no ejecuta el trabajo directamente: lo lanza con `launch` y guarda el `Job` en una variable. En el siguiente tick, `job.isActive` te dice si sigue corriendo.

<details>
<summary>Solución</summary>

```kotlin
import kotlinx.coroutines.*
import kotlin.time.Duration
import kotlin.time.Duration.Companion.milliseconds

fun CoroutineScope.lanzarFrecuenciaFija(
  intervalo: Duration,
  trabajo: suspend () -> Unit,
): Job = launch {
  var actual: Job? = null
  var tick = 0
  while (isActive) {
    if (actual?.isActive == true) {
      println("tick $tick: omitido, la anterior sigue corriendo")
    } else {
      println("tick $tick: arranca")
      actual = launch { trabajo() }
    }
    tick++
    delay(intervalo)
  }
}

fun main() = runBlocking {
  val job = lanzarFrecuenciaFija(100.milliseconds) { delay(250) }
  delay(550)
  job.cancelAndJoin()
  println("fin")
}
```

Salida:

```
tick 0: arranca
tick 1: omitido, la anterior sigue corriendo
tick 2: omitido, la anterior sigue corriendo
tick 3: arranca
tick 4: omitido, la anterior sigue corriendo
tick 5: omitido, la anterior sigue corriendo
fin
```

El trabajo es **hija** del job del bucle, así que al cancelar el bucle también se cancela la ejecución que estaba en curso (regla 2 de la lección 3). Compáralo con el bucle simple de la lección 4: aquí escribiste bastante más código para conseguir algo que el fixed delay te da gratis. Por eso la lección recomienda fixed delay para casi todo. Y un detalle: como `delay` cuenta desde que termina cada vuelta, los ticks se van corriendo unos milisegundos. Para horas exactas de verdad se calcula la próxima hora contra el reloj.

</details>

-----

## Checkpoint

Responde sin mirar las lecciones. Si dudas en alguna, vuelve a la lección indicada.

<details>
<summary>1. ¿Qué diferencia hay entre <code>Thread.sleep(200)</code> y <code>delay(200)</code>?</summary>

`Thread.sleep` bloquea el hilo entero: nadie más puede usarlo durante esos 200 ms. `delay` pausa solo la corrutina y el hilo queda libre para atender otras. Ver [`suspend`](01-suspend.md).

</details>

<details>
<summary>2. Si marcas una función como <code>suspend</code>, ¿corre en otro hilo? ¿Deja de bloquear?</summary>

Ninguna de las dos. `suspend` solo dice que la función **puede** pausarse. En qué hilo corre lo decide el dispatcher, y si adentro llamas a algo bloqueante (JDBC, `Thread.sleep`), sigue bloqueando. Ver [`suspend`](01-suspend.md).

</details>

<details>
<summary>3. ¿Por qué no se usa <code>runBlocking</code> dentro de una ruta de Ktor?</summary>

Porque bloquea el hilo hasta que termina todo lo de adentro, y los hilos que atienden peticiones son pocos. Es la puerta de entrada en `main` y en tests, no dentro de código que ya es asíncrono. Ver [`suspend`](01-suspend.md).

</details>

<details>
<summary>4. ¿Por qué <code>val a = async { x() }.await(); val b = async { y() }.await()</code> no es paralelo?</summary>

Porque cada `await()` espera en el acto, antes de lanzar el siguiente `async`. Primero se lanzan todos y después se esperan. Ver [Scopes y dispatchers](02-scopes-y-dispatchers.md).

</details>

<details>
<summary>5. ¿En qué dispatcher corres una consulta JDBC y por qué?</summary>

En `Dispatchers.IO`, con `withContext`. JDBC bloquea el hilo mientras espera a la base. En el pool de IO ese bloqueo no deja sin hilos a las peticiones. Ver [Scopes y dispatchers](02-scopes-y-dispatchers.md).

</details>

<details>
<summary>6. ¿Cuándo usas <code>coroutineScope</code> y cuándo <code>supervisorScope</code>?</summary>

`coroutineScope` cuando las tareas forman un solo resultado: si una falla, no tiene sentido seguir con las demás. `supervisorScope` cuando son independientes, como sincronizar varios proveedores, y cada hija maneja su propio error. Ver [Concurrencia estructurada](03-concurrencia-estructurada.md).

</details>

<details>
<summary>7. ¿Por qué <code>catch (e: Exception)</code> rompe la cancelación y cómo se arregla?</summary>

Porque `CancellationException` hereda de `Exception`, así que ese `catch` también la atrapa y la corrutina sigue como si nada. Se arregla con un `catch (e: CancellationException) { throw e }` **antes** del general. Lo mismo vale para `runCatching`. Ver [Concurrencia estructurada](03-concurrencia-estructurada.md).

</details>

<details>
<summary>8. Tienes 3 instancias del backend y un job que manda un reporte por email. ¿Qué pasa y qué haces?</summary>

Cada instancia corre sus propios jobs, así que el reporte llega 3 veces. Opciones: un lease en la base para que solo una instancia trabaje, una sola instancia con los jobs habilitados, o trabajo idempotente (que con emails no siempre es posible). Ver [Jobs programados](04-jobs-programados.md).

</details>

-----

Vuelve al índice de la carpeta: [README](README.md).
