# Funciones `inline`

## En una frase

Una función `inline` le pide al compilador que **copie su cuerpo y el de sus lambdas** en cada lugar donde se llama; así las lambdas no cuestan nada, un `return` dentro de ellas puede salir de la función de afuera, y los tipos genéricos se pueden consultar en tiempo de ejecución con `reified`.

-----

## Antes de empezar

Conviene que ya sepas:

* Funciones de orden superior y closures: [Funciones de orden superior](01-funciones-de-orden-superior.md).

Palabras nuevas (también están en el [Glosario](Glosario.md)):

* **`inline`:** Modificador que hace que el compilador copie el cuerpo de la función en cada llamada.
* **`return` no local:** Un `return` dentro de una lambda que sale de la función que la contiene, no solo de la lambda.
* **Etiqueta (label):** Nombre que marca una lambda, para hacer `return@nombre` desde adentro.
* **Borrado de tipos (type erasure):** En la JVM, los tipos genéricos desaparecen al ejecutar: una `List<String>` es solo una `List`.
* **`reified`:** Modificador que hace que un tipo genérico de una función `inline` siga disponible en tiempo de ejecución.

-----

## El costo de una lambda

En la JVM, cada lambda que **captura** variables se convierte en un objeto. Si llamas a una función de orden superior dentro de un bucle de un millón de vueltas, puedes crear un millón de objetos y además pagar una llamada extra por cada uno.

```kotlin
fun <T> medir(bloque: () -> T): T {
  val inicio = System.nanoTime()
  val resultado = bloque()
  println("Tardó ${System.nanoTime() - inicio} ns")
  return resultado
}
```

## `inline`: copiar en vez de llamar

```kotlin
inline fun <T> medir(bloque: () -> T): T { /* lo mismo */ }

val total = medir { calcularTotal() }
```

El compilador genera, en el lugar de la llamada, algo equivalente a:

```kotlin
val inicio = System.nanoTime()
val resultado = calcularTotal()   // el cuerpo de la lambda, pegado acá
println("Tardó ${System.nanoTime() - inicio} ns")
val total = resultado
```

No hay objeto para la lambda ni llamada extra. Por eso `filter`, `map`, `let`, `apply` y casi todas las funciones de la librería estándar que reciben lambdas son `inline`: usarlas cuesta lo mismo que escribir el bucle a mano.

## El efecto más visible: `return` no local

Como la lambda se copia **dentro** de la función que la llama, un `return` en la lambda sale de **esa** función:

```kotlin
fun buscarPrimeroCaro(pedidos: List<Pedido>): Pedido? {
  pedidos.forEach {
    if (it.total > 1000) return it   // sale de buscarPrimeroCaro, no solo de la lambda
  }
  return null
}
```

Esto funciona porque `forEach` es `inline`. Con una función que **no** es inline, un `return` sin etiqueta dentro de la lambda no compila: la lambda podría ejecutarse más tarde, cuando la función de afuera ya terminó.

### Volver solo de la lambda: etiquetas

Si quieres salir **solo** de la lambda (como un `continue`), usa una etiqueta:

```kotlin
pedidos.forEach {
  if (it.total <= 0) return@forEach   // salta este elemento y sigue con el siguiente
  procesar(it)
}
```

Esta es una fuente clásica de bugs: escribir `return` pensando que es un `continue`, y salir de toda la función.

## `noinline` y `crossinline`

Si una función `inline` recibe **varias** lambdas y quieres guardar una para después (en una variable, o pasarla a una función que no es inline), esa no se puede copiar. Se marca con `noinline`:

```kotlin
inline fun ejecutar(antes: () -> Unit, noinline despues: () -> Unit) {
  antes()                     // se copia
  programarParaLuego(despues) // se guarda: no se puede copiar
}
```

`crossinline` se usa cuando la lambda se copia, pero se ejecuta dentro de **otro** contexto (otra lambda u objeto). Prohíbe el `return` no local, porque ahí no tendría sentido:

```kotlin
inline fun enHiloAparte(crossinline bloque: () -> Unit) {
  Thread { bloque() }.start()  // `return` en `bloque` no podría salir de quien llamó
}
```

## `reified`: tipos genéricos en tiempo de ejecución

En la JVM, los tipos genéricos se **borran** al ejecutar. Esta función no compila:

```kotlin
fun <T> esDelTipo(valor: Any): Boolean = valor is T   // ❌ T no existe en tiempo de ejecución
```

En una función `inline`, el compilador copia el cuerpo en cada llamada y **sabe** cuál es `T` en ese punto. Con `reified` lo puedes usar:

```kotlin
inline fun <reified T> esDelTipo(valor: Any): Boolean = valor is T

esDelTipo<String>("hola") // true
esDelTipo<Int>("hola")    // false
```

Así funcionan muchas APIs que ya usaste:

| Llamada | Por qué necesita `reified` |
| --- | --- |
| `call.receive<CrearPedidoRequest>()` | Ktor necesita saber a qué clase convertir el JSON |
| `get<PedidoRepository>()`, `inject<CrearPedido>()` | Koin busca la definición por tipo |
| `exception<NoAutenticadoException> { ... }` | StatusPages registra el handler para ese tipo |
| `Json.decodeFromString<Pedido>(texto)` | kotlinx.serialization busca el serializador del tipo |

Sin `reified`, tendrías que escribir `call.receive(CrearPedidoRequest::class)`.

## Cuándo marcar tus funciones como `inline`

* ✅ Funciones **chicas** que reciben lambdas y se llaman mucho (utilidades como `medir`, `conTransaccion`, `reintentar`).
* ✅ Cuando necesitas `reified`.
* ❌ Funciones grandes: el cuerpo se copia en **cada** llamada y el código compilado crece.
* ❌ Funciones que no reciben lambdas ni usan `reified`: no ganas nada, y el compilador te avisa.

En la práctica, vas a **usar** muchas funciones inline y vas a **escribir** pocas.

-----

## Resumen en 5 líneas

1. Una lambda que captura variables suele crear un objeto; `inline` copia la función y sus lambdas en cada llamada y elimina ese costo.
2. Por eso las funciones de colecciones y de alcance son `inline` y cuestan lo mismo que un bucle.
3. En una lambda inline, `return` sale de la función de afuera; para salir solo de la lambda, `return@etiqueta`.
4. `noinline` permite guardar una lambda; `crossinline` prohíbe el `return` no local cuando se ejecuta en otro contexto.
5. `reified` mantiene el tipo genérico en tiempo de ejecución: por eso existen `call.receive<T>()` y `get<T>()`.

-----

## Siguiente lección

Terminaste la programación funcional. Lo próximo es [Corrutinas](../10-corrutinas/README.md). Vuelve al índice: [README principal](../README.md).
