# Ejercicios de programación funcional

Cuaderno de práctica de toda la carpeta: funciones de orden superior, closures, colecciones, referencias, lambdas con receptor, funciones de alcance, `inline` y callbacks. Cada solución es un programa completo con `fun main()`: puedes pegarlo en un archivo `.kt` y ejecutarlo.

Requisitos previos: [Funciones de orden superior](01-funciones-de-orden-superior.md), [Funciones de alcance](02-funciones-de-alcance.md), [`inline`](03-inline.md) y [Callbacks](04-callbacks.md).

**Cómo usarlo:** intenta cada ejercicio **antes** de abrir la solución. Si te trabas, vuelve a la lección, no a la solución. Mirar la respuesta antes de pelearla es leer, no entrenar.

Casi todos los ejercicios usan estos datos:

```kotlin
data class Pedido(val id: String, val pais: String, val total: Double)

val pedidos = listOf(
  Pedido("p1", "AR", 1500.0),
  Pedido("p2", "UY", 300.0),
  Pedido("p3", "AR", 800.0),
  Pedido("p4", "CL", 2200.0),
  Pedido("p5", "UY", 950.0),
)
```

-----

## Ejercicios guiados

### Ejercicio 1: tu propio `filtrar`

**Objetivo:** escribir una función que recibe otra función como parámetro.

**Contexto:** tienes que sacar los pedidos grandes, los de Uruguay y los argentinos baratos. Sin funciones de orden superior escribirías tres bucles casi iguales.

**Instrucciones:**

1. Escribe `filtrar(pedidos: List<Pedido>, condicion: (Pedido) -> Boolean): List<Pedido>` con un bucle `for`. Nada de usar `filter` adentro.
2. Úsala con trailing lambda para obtener:
   * los pedidos con total mayor a 1000,
   * los de `"UY"`,
   * los de `"AR"` con total menor a 1000.
3. Imprime solo los ids de cada resultado.

<details>
<summary>Solución</summary>

```kotlin
data class Pedido(val id: String, val pais: String, val total: Double)

fun filtrar(pedidos: List<Pedido>, condicion: (Pedido) -> Boolean): List<Pedido> {
  val resultado = mutableListOf<Pedido>()
  for (p in pedidos) if (condicion(p)) resultado.add(p)
  return resultado
}

fun main() {
  val pedidos = listOf(
    Pedido("p1", "AR", 1500.0),
    Pedido("p2", "UY", 300.0),
    Pedido("p3", "AR", 800.0),
    Pedido("p4", "CL", 2200.0),
    Pedido("p5", "UY", 950.0),
  )

  println(filtrar(pedidos) { it.total > 1000 }.map { it.id })
  println(filtrar(pedidos) { it.pais == "UY" }.map { it.id })
  println(filtrar(pedidos) { it.pais == "AR" && it.total < 1000 }.map { it.id })
}
```

Salida:

```
[p1, p4]
[p2, p5]
[p3]
```

</details>

**Qué observar:** el bucle se escribió **una vez**. Lo único que cambia entre las tres llamadas es la lambda. Eso es exactamente lo que hace `filter` de la librería estándar: acabas de escribir tu versión.

-----

### Ejercicio 2: lee el tipo

**Objetivo:** leer tipos de función sin ejecutar nada.

**Contexto:** si no puedes leer `(Double) -> (Pedido) -> Boolean`, los DSL de Ktor y Koin te van a parecer magia.

**Instrucciones:** escribe en un papel el tipo de cada expresión, o lo que imprime. Después compruébalo con la solución.

```kotlin
fun mayorQue(minimo: Double): (Pedido) -> Boolean = { it.total > minimo }

val doble = { x: Int -> x * 2 }                          // a) tipo de doble
val saludar = { println("hola") }                        // b) tipo de saludar
val sumar = { a: Double, b: Double -> a + b }            // c) tipo de sumar
val esGrande = mayorQue(1000.0)                          // d) tipo de esGrande
val fabrica = ::mayorQue                                 // e) tipo de fabrica
println(mayorQue(100.0)(Pedido("p9", "AR", 50.0)))       // f) qué imprime
```

<details>
<summary>Solución</summary>

| | Respuesta | Por qué |
| --- | --- | --- |
| a) | `(Int) -> Int` | Recibe un `Int` y `x * 2` es un `Int` |
| b) | `() -> Unit` | No recibe nada; `println` devuelve `Unit` |
| c) | `(Double, Double) -> Double` | Dos parámetros, la suma es `Double` |
| d) | `(Pedido) -> Boolean` | Es **lo que devuelve** `mayorQue`, no `mayorQue` |
| e) | `(Double) -> (Pedido) -> Boolean` | La función entera: recibe un `Double` y devuelve una función |
| f) | `false` | El primer `()` le pasa `100.0` a `mayorQue`; el segundo le pasa el pedido a la lambda devuelta. `50.0 > 100.0` es `false` |

</details>

**Qué observar:** la d) y la e) son la confusión típica. `mayorQue(1000.0)` **llama** a la función y te da la lambda. `::mayorQue` **no la llama**: te da la función misma. La flecha se lee de izquierda a derecha: `(Double) -> ((Pedido) -> Boolean)`.

-----

### Ejercicio 3: fábrica de filtros

**Objetivo:** devolver funciones y combinarlas (closures).

**Contexto:** con `mayorQue` ya tienes un filtro configurable. Ahora quieres armar condiciones más complejas sin escribir lambdas nuevas cada vez.

**Instrucciones:**

1. Escribe `delPais(pais: String): (Pedido) -> Boolean`.
2. Escribe `y(a, b)`, que recibe dos condiciones y devuelve una que se cumple si se cumplen **las dos**.
3. Escribe `no(condicion)`, que devuelve la condición contraria.
4. Usando solo `mayorQue`, `delPais`, `y` y `no` dentro de `filter`, obtén:
   * argentinos de más de 1000,
   * todos los que **no** son de Argentina,
   * los que no son de Argentina y superan 500.

<details>
<summary>Solución</summary>

```kotlin
data class Pedido(val id: String, val pais: String, val total: Double)

fun mayorQue(minimo: Double): (Pedido) -> Boolean = { it.total > minimo }
fun delPais(pais: String): (Pedido) -> Boolean = { it.pais == pais }

fun y(a: (Pedido) -> Boolean, b: (Pedido) -> Boolean): (Pedido) -> Boolean =
  { a(it) && b(it) }

fun no(condicion: (Pedido) -> Boolean): (Pedido) -> Boolean =
  { !condicion(it) }

fun main() {
  val pedidos = listOf(
    Pedido("p1", "AR", 1500.0),
    Pedido("p2", "UY", 300.0),
    Pedido("p3", "AR", 800.0),
    Pedido("p4", "CL", 2200.0),
    Pedido("p5", "UY", 950.0),
  )

  println(pedidos.filter(y(delPais("AR"), mayorQue(1000.0))).map { it.id })
  println(pedidos.filter(no(delPais("AR"))).map { it.id })
  println(pedidos.filter(y(no(delPais("AR")), mayorQue(500.0))).map { it.id })
}
```

Salida:

```
[p1]
[p2, p4, p5]
[p4, p5]
```

</details>

**Qué observar:** `y` recibe dos funciones y devuelve una tercera que **recuerda** a `a` y `b`. Es la misma closure que `mayorQue` con `minimo`, pero capturando funciones en vez de números. Fíjate que `filter(...)` lleva paréntesis y no llaves: le pasas una función que ya existe, no una lambda nueva.

-----

### Ejercicio 4: un contador con memoria

**Objetivo:** ver que una closure puede capturar una variable `var` y modificarla.

**Contexto:** en el ejercicio anterior las closures solo **leían** lo que capturaban. ¿Qué pasa si lo cambian?

**Instrucciones:**

1. Escribe `crearContador(): () -> Int`. Adentro declara `var cuenta = 0` y devuelve una lambda que suma 1 a `cuenta` y la devuelve.
2. **Antes de ejecutar**, predice qué imprime este código:

```kotlin
val a = crearContador()
val b = crearContador()
println(a())
println(a())
println(b())
println(a())
```

<details>
<summary>Solución</summary>

```kotlin
fun crearContador(): () -> Int {
  var cuenta = 0
  return {
    cuenta += 1
    cuenta
  }
}

fun main() {
  val a = crearContador()
  val b = crearContador()
  println(a())
  println(a())
  println(b())
  println(a())
}
```

Salida:

```
1
2
1
3
```

</details>

**Qué observar:** `crearContador` ya terminó, pero `cuenta` sigue viva porque la lambda la capturó. Y cada llamada a `crearContador()` crea **su propia** `cuenta`: por eso `b()` empieza en 1 y no afecta a `a`. Si vienes de JavaScript, es idéntico a una closure de JS. En la JVM, Kotlin envuelve esa `var` en un objeto para que la lambda la pueda modificar.

-----

### Ejercicio 5: reporte sin bucles

**Objetivo:** resolver consultas con las operaciones de colecciones.

**Contexto:** te piden un reporte de ventas. Prohibido escribir `for`.

**Instrucciones:** con la lista `pedidos`, obtén:

1. Los ids de los pedidos de Uruguay.
2. El total facturado.
3. Si hay algún pedido de Brasil (`"BR"`).
4. El total por país, como un mapa.
5. Los ids de los 2 pedidos más caros.
6. El id del primer pedido de Brasil, o el texto `"sin pedidos de BR"` si no hay.
7. Dos listas separadas: los de más de 1000 y el resto.

<details>
<summary>Solución</summary>

```kotlin
data class Pedido(val id: String, val pais: String, val total: Double)

fun main() {
  val pedidos = listOf(
    Pedido("p1", "AR", 1500.0),
    Pedido("p2", "UY", 300.0),
    Pedido("p3", "AR", 800.0),
    Pedido("p4", "CL", 2200.0),
    Pedido("p5", "UY", 950.0),
  )

  println(pedidos.filter { it.pais == "UY" }.map { it.id })                     // 1
  println(pedidos.sumOf { it.total })                                           // 2
  println(pedidos.any { it.pais == "BR" })                                      // 3

  val totalPorPais = pedidos
    .groupBy { it.pais }
    .mapValues { (_, delPais) -> delPais.sumOf { it.total } }
  println(totalPorPais)                                                         // 4

  println(pedidos.sortedByDescending { it.total }.take(2).map { it.id })        // 5
  println(pedidos.firstOrNull { it.pais == "BR" }?.id ?: "sin pedidos de BR")   // 6

  val (caros, resto) = pedidos.partition { it.total > 1000 }
  println("caros=${caros.map { it.id }} resto=${resto.map { it.id }}")          // 7
}
```

Salida:

```
[p2, p5]
5750.0
false
{AR=2300.0, UY=1250.0, CL=2200.0}
[p4, p1]
sin pedidos de BR
caros=[p1, p4] resto=[p2, p3, p5]
```

</details>

**Qué observar:**

* En la 4, `groupBy` te da `Map<String, List<Pedido>>` y `mapValues` transforma cada lista en su suma. El `(_, delPais)` desarma cada entrada del mapa e ignora la clave con `_`.
* En la 6, `first { }` habría lanzado `NoSuchElementException`. `firstOrNull` + `?.` + `?:` es la combinación segura.
* En la 7, `partition` devuelve un `Pair` y `val (caros, resto)` lo desarma en dos variables.

-----

### Ejercicio 6: de lambda a referencia

**Objetivo:** reconocer cuándo una lambda se puede reemplazar por `::`.

**Contexto:** una lambda que solo llama a una función que ya existe es ruido.

**Instrucciones:** reescribe cada línea con una referencia a función. Una de ellas **no** se puede: encuéntrala y explica por qué.

```kotlin
data class PedidoId(val valor: String)
fun esValido(p: Pedido) = p.total > 0

pedidos.map { it.id }                      // a
pedidos.filter { esValido(it) }            // b
listOf("p1", "p2").map { PedidoId(it) }    // c
pedidos.forEach { println(it) }            // d
pedidos.filter { it.total > 1000 }         // e
```

<details>
<summary>Solución</summary>

```kotlin
data class Pedido(val id: String, val pais: String, val total: Double)
data class PedidoId(val valor: String)

fun esValido(p: Pedido) = p.total > 0

fun main() {
  val pedidos = listOf(Pedido("p1", "AR", 1500.0), Pedido("p2", "UY", 300.0))

  println(pedidos.map(Pedido::id))                 // a: propiedad como función
  println(pedidos.filter(::esValido).size)         // b: función de nivel superior
  println(listOf("p1", "p2").map(::PedidoId))      // c: un constructor es una función
  pedidos.forEach(::println)                       // d
  println(pedidos.filter { it.total > 1000 }.size) // e: se queda como lambda
}
```

Salida:

```
[p1, p2]
2
[PedidoId(valor=p1), PedidoId(valor=p2)]
Pedido(id=p1, pais=AR, total=1500.0)
Pedido(id=p2, pais=UY, total=300.0)
1
```

</details>

**Qué observar:** la e) no tiene referencia porque `it.total > 1000` no es "llamar a una función con `it`": es una expresión. Podrías crear `fun esCaro(p: Pedido) = p.total > 1000` y pasar `::esCaro`, pero solo vale la pena si ese criterio tiene nombre en tu negocio y se repite. Una referencia no es "mejor" que una lambda: es más corta cuando ya existe la función.

-----

### Ejercicio 7: tu primer DSL

**Objetivo:** escribir una función que recibe una lambda con receptor.

**Contexto:** quieres armar un carrito así, sin repetir `carrito.` en cada línea:

```kotlin
val c = carrito {
  item("Teclado", 1, 45.0)
  item("Mouse", 2, 15.5)
}
```

**Instrucciones:**

1. Crea `data class Linea(val nombre: String, val cantidad: Int, val precio: Double)`.
2. Crea la clase `Carrito` con una lista privada de líneas, una función `item(...)` que agrega una línea, `total()` y `resumen()`.
3. Escribe `fun carrito(construir: Carrito.() -> Unit): Carrito`.
4. Imprime el resumen y el total.

<details>
<summary>Solución</summary>

```kotlin
data class Linea(val nombre: String, val cantidad: Int, val precio: Double) {
  val subtotal: Double get() = cantidad * precio
}

class Carrito {
  private val lineas = mutableListOf<Linea>()

  fun item(nombre: String, cantidad: Int, precio: Double) {
    lineas.add(Linea(nombre, cantidad, precio))
  }

  fun total(): Double = lineas.sumOf { it.subtotal }

  fun resumen(): String =
    lineas.joinToString("\n") { "${it.cantidad} x ${it.nombre} = ${it.subtotal}" }
}

fun carrito(construir: Carrito.() -> Unit): Carrito {
  val c = Carrito()
  c.construir()   // ejecuta el bloque con c como this
  return c
}

fun main() {
  val c = carrito {
    item("Teclado", 1, 45.0)
    item("Mouse", 2, 15.5)
  }
  println(c.resumen())
  println("Total: ${c.total()}")
}
```

Salida:

```
1 x Teclado = 45.0
2 x Mouse = 31.0
Total: 76.0
```

</details>

**Qué observar:** dentro de las llaves, `item(...)` es `this.item(...)`, y `this` es el `Carrito` que crea `carrito`. Es lo mismo que pasa en `routing { get("/") { } }`. Y mira esto: `fun carrito(construir: Carrito.() -> Unit) = Carrito().apply(construir)` hace exactamente lo mismo en una línea, porque `apply` recibe justamente un `T.() -> Unit`.

-----

### Ejercicio 8: la función de alcance equivocada

**Objetivo:** detectar errores de `apply`, `let` y `also` por lo que **devuelven**.

**Contexto:** las cuatro funciones compilan. Tres hacen algo distinto de lo que su autor quería.

**Instrucciones:** para cada caso, predice qué imprime, di qué está mal y corrígelo.

```kotlin
// a) quería el email sin espacios
val limpio = "  ana@mail.com  ".apply { trim() }
println("[$limpio]")

// b) quería loguear el pedido y seguir usándolo
val pedido = Pedido("p1", "AR", 1500.0).let { println("Creado ${it.id}") }
println(pedido)

// c) quería configurar una conexión
class Conexion { var host = ""; var puerto = 0 }
val conexion = Conexion().let {
  it.host = "localhost"
  it.puerto = 5432
}
println(conexion)

// d) página de un query param: si no es un número positivo, 1
println("0".toIntOrNull()?.takeIf { it > 0 } ?: 1)
```

<details>
<summary>Solución</summary>

Lo que imprime el código original:

```
[  ana@mail.com  ]
Creado p1
kotlin.Unit
kotlin.Unit
1
```

* **a)** `apply` devuelve el `String` original y `trim()` devuelve uno nuevo que se pierde. Corrección: `"  ana@mail.com  ".trim()`.
* **b)** `let` devuelve el resultado del bloque, y `println` devuelve `Unit`. `pedido` ya no es un `Pedido`. Corrección: `also`, que devuelve el objeto.
* **c)** Igual que b): la última línea del bloque es una asignación, y `let` devuelve `Unit`. Para configurar propiedades, `apply`.
* **d)** Está bien. `"0"` se convierte en `0`, `takeIf` lo rechaza y da `null`, y el Elvis pone `1`.

Corregido:

```kotlin
data class Pedido(val id: String, val pais: String, val total: Double)

class Conexion {
  var host = ""
  var puerto = 0
  override fun toString() = "Conexion($host:$puerto)"
}

fun main() {
  val limpio = "  ana@mail.com  ".trim()
  println("[$limpio]")

  val pedido = Pedido("p1", "AR", 1500.0).also { println("Creado ${it.id}") }
  println(pedido)

  val conexion = Conexion().apply {
    host = "localhost"
    puerto = 5432
  }
  println(conexion)

  for (texto in listOf("0", "abc", "3")) {
    println(texto.toIntOrNull()?.takeIf { it > 0 } ?: 1)
  }
}
```

Salida:

```
[ana@mail.com]
Creado p1
Pedido(id=p1, pais=AR, total=1500.0)
Conexion(localhost:5432)
1
1
3
```

</details>

**Qué observar:** los tres errores se detectan con **una sola pregunta**: ¿qué quiero que devuelva? Si es el objeto, `apply` o `also`. Si es otra cosa, `let` o `run`. El compilador no te avisa: `kotlin.Unit` en la salida es la pista.

-----

### Ejercicio 9: `return` que se escapa

**Objetivo:** distinguir `return` de `return@forEach` en una lambda inline.

**Contexto:** alguien quiso saltar los totales negativos y escribió `return` pensando que era un `continue`.

**Instrucciones:** predice la salida de `main` y después corrige `procesar` para que salte los negativos, procese el resto e imprima `fin`.

```kotlin
fun primeroCaro(pedidos: List<Pedido>): String {
  pedidos.forEach {
    if (it.total > 1000) return it.id
  }
  return "ninguno"
}

fun procesar(totales: List<Double>) {
  totales.forEach {
    if (it < 0) return
    println("procesado $it")
  }
  println("fin")
}

fun main() {
  println(primeroCaro(listOf(Pedido("p2", "UY", 300.0), Pedido("p1", "AR", 1500.0))))
  procesar(listOf(100.0, -5.0, 200.0))
}
```

<details>
<summary>Solución</summary>

Salida del código original:

```
p1
procesado 100.0
```

En `primeroCaro`, el `return` sale de la función: es justo lo que se quería. En `procesar`, el mismo `return` también sale de **toda** la función al ver `-5.0`: no procesa `200.0` ni imprime `fin`.

Corrección:

```kotlin
fun procesar(totales: List<Double>) {
  totales.forEach {
    if (it < 0) return@forEach   // sale solo de esta vuelta de la lambda
    println("procesado $it")
  }
  println("fin")
}
```

Salida:

```
procesado 100.0
procesado 200.0
fin
```

</details>

**Qué observar:** el mismo `return` es correcto en una función y un bug en la otra. Esto solo compila porque `forEach` es `inline`: la lambda se copia dentro de la función y el `return` sale de ella. Si te cuesta leerlo, un `for` común con `continue` es igual de válido.

-----

### Ejercicio 10: `reified` en acción

**Objetivo:** escribir una función genérica que pregunta por el tipo en tiempo de ejecución.

**Contexto:** tienes una `List<Any>` mezclada y quieres sacar solo los elementos de un tipo.

**Instrucciones:**

1. Escribe una extension function `soloDeTipo<T>()` sobre `List<Any>` que devuelva `List<T>` usando `is T`.
2. Prueba primero **sin** `inline` ni `reified` y lee el error del compilador.
3. Arréglala y úsala con `String` e `Int` sobre `listOf("hola", 1, 2.5, "chau", 3)`.

<details>
<summary>Solución</summary>

Sin `reified`, `x is T` da el error *Cannot check for instance of erased type: T*: en la JVM, `T` ya no existe al ejecutar.

```kotlin
inline fun <reified T> List<Any>.soloDeTipo(): List<T> {
  val resultado = mutableListOf<T>()
  for (x in this) if (x is T) resultado.add(x)
  return resultado
}

fun main() {
  val mezcla: List<Any> = listOf("hola", 1, 2.5, "chau", 3)
  println(mezcla.soloDeTipo<String>())
  println(mezcla.soloDeTipo<Int>())
}
```

Salida:

```
[hola, chau]
[1, 3]
```

</details>

**Qué observar:** la librería estándar ya trae esta función: `filterIsInstance<String>()`. Y es `inline` con `reified` por la misma razón que la tuya. Es el mismo truco que usan `call.receive<T>()` en Ktor y `get<T>()` en Koin.

-----

### Ejercicio 11: pagos con callbacks

**Objetivo:** diseñar una función con dos callbacks y usar `_`.

**Contexto:** un pago puede aprobarse o rechazarse por dos motivos distintos. La función que decide no debe saber si quien llama muestra un mensaje, reintenta o guarda un log.

**Instrucciones:**

1. Escribe `procesarPago(monto, saldo, onAprobado, onRechazado)`:
   * `onAprobado: (Double) -> Unit` recibe el saldo restante.
   * `onRechazado: (String, Double) -> Unit` recibe el motivo y cuánto falta.
2. Rechaza si el monto es `<= 0` (motivo `"Monto inválido"`) o si supera el saldo (`"Saldo insuficiente"`).
3. Recorre los montos `300.0`, `1500.0` y `-10.0` con saldo `1000.0` e imprime un mensaje para cada uno. En el rechazo no uses el segundo parámetro.

<details>
<summary>Solución</summary>

```kotlin
fun procesarPago(
  monto: Double,
  saldo: Double,
  onAprobado: (Double) -> Unit,
  onRechazado: (String, Double) -> Unit,
) {
  when {
    monto <= 0 -> onRechazado("Monto inválido", 0.0)
    monto > saldo -> onRechazado("Saldo insuficiente", monto - saldo)
    else -> onAprobado(saldo - monto)
  }
}

fun main() {
  for (monto in listOf(300.0, 1500.0, -10.0)) {
    procesarPago(
      monto,
      saldo = 1000.0,
      onAprobado = { restante -> println("$monto aprobado. Saldo restante: $restante") },
      onRechazado = { motivo, _ -> println("$monto rechazado: $motivo") },
    )
  }
}
```

Salida:

```
300.0 aprobado. Saldo restante: 700.0
1500.0 rechazado: Saldo insuficiente
-10.0 rechazado: Monto inválido
```

</details>

**Qué observar:** `onRechazado` tiene dos parámetros, así que no hay `it`: hay que nombrarlos, y `_` dice "este lo ignoro a propósito". Además, las lambdas usan `monto`, que no es parámetro de ellas: lo **capturan** del `for`. Closures otra vez.

-----

## Retos

### Reto 1: tus propios `filter`, `map` y `fold`

**Misión:** escribe `miFilter`, `miMap` y `miFold` como extension functions **genéricas** sobre `List<T>`, marcadas como `inline`. Pruébalas con:

```kotlin
listOf(1, 2, 3, 4).miFilter { it % 2 == 0 }.miMap { it * 10 }   // [20, 40]
listOf(1, 2, 3, 4).miFold(0) { acumulado, x -> acumulado + x }   // 10
pedidos.miFold(0.0) { acumulado, p -> acumulado + p.total }      // 5750.0
```

**Pista:** `miMap` necesita **dos** genéricos, `<T, R>`, porque transforma un tipo en otro. `miFold` también: el tipo del acumulado no tiene por qué ser el de los elementos.

<details>
<summary>Solución</summary>

```kotlin
data class Pedido(val id: String, val pais: String, val total: Double)

inline fun <T> List<T>.miFilter(condicion: (T) -> Boolean): List<T> {
  val resultado = mutableListOf<T>()
  for (x in this) if (condicion(x)) resultado.add(x)
  return resultado
}

inline fun <T, R> List<T>.miMap(transformar: (T) -> R): List<R> {
  val resultado = mutableListOf<R>()
  for (x in this) resultado.add(transformar(x))
  return resultado
}

inline fun <T, A> List<T>.miFold(inicial: A, operar: (A, T) -> A): A {
  var acumulado = inicial
  for (x in this) acumulado = operar(acumulado, x)
  return acumulado
}

fun main() {
  val pedidos = listOf(
    Pedido("p1", "AR", 1500.0),
    Pedido("p2", "UY", 300.0),
    Pedido("p3", "AR", 800.0),
    Pedido("p4", "CL", 2200.0),
    Pedido("p5", "UY", 950.0),
  )

  println(listOf(1, 2, 3, 4).miFilter { it % 2 == 0 }.miMap { it * 10 })
  println(listOf(1, 2, 3, 4).miFold(0) { acumulado, x -> acumulado + x })
  println(pedidos.miFold(0.0) { acumulado, p -> acumulado + p.total })
}
```

Salida:

```
[20, 40]
10
5750.0
```

Si abres el código fuente de `filter` en IntelliJ (Ctrl + clic), vas a ver casi lo mismo: `inline`, un bucle y una lista nueva.

</details>

-----

### Reto 2: memoizar

**Misión:** escribe `memoizar(f)`, que recibe una función `(A) -> R` y devuelve **otra** función del mismo tipo que guarda los resultados: si la llamas dos veces con el mismo argumento, la segunda vez no ejecuta `f`. Pruébala con un cálculo que imprima `calculando...` cada vez que realmente se ejecuta.

**Pista:** la caché es un `mutableMapOf<A, R>()` que vive **dentro** de `memoizar` y que la lambda devuelta captura. Mira `getOrPut` en los mapas.

<details>
<summary>Solución</summary>

```kotlin
fun <A, R> memoizar(f: (A) -> R): (A) -> R {
  val cache = mutableMapOf<A, R>()
  return { a -> cache.getOrPut(a) { f(a) } }
}

fun main() {
  val conImpuestos = memoizar { precio: Double ->
    println("calculando $precio...")
    precio * 1.21
  }

  println(conImpuestos(100.0))
  println(conImpuestos(100.0))
  println(conImpuestos(200.0))
}
```

Salida:

```
calculando 100.0...
121.0
121.0
calculando 200.0...
242.0
```

La segunda llamada con `100.0` no imprime `calculando`: la saca de la caché. Es el contador del ejercicio 4, pero capturando un mapa. Ojo: esta caché crece sin límite y no es segura con varios hilos; sirve para entender la idea, no para producción.

</details>

-----

### Reto 3: reglas de validación como datos

**Misión:** valida un pedido con una **lista de reglas**, donde cada regla es una función que devuelve un mensaje de error o `null` si todo está bien. Escribe `validar(pedido, reglas): List<String>` con las reglas: total positivo, país en `AR`, `UY` o `CL`, e id no vacío. Prueba con un pedido válido y con `Pedido("", "BR", -5.0)`.

**Pista:** declara `typealias Regla = (Pedido) -> String?` para no repetir el tipo. Para quedarte solo con los errores, busca `mapNotNull`.

<details>
<summary>Solución</summary>

```kotlin
data class Pedido(val id: String, val pais: String, val total: Double)

typealias Regla = (Pedido) -> String?

val reglas = listOf<Regla>(
  { if (it.total <= 0) "El total debe ser positivo" else null },
  { if (it.pais !in setOf("AR", "UY", "CL")) "No enviamos a ${it.pais}" else null },
  { if (it.id.isBlank()) "Falta el id" else null },
)

fun validar(pedido: Pedido, reglas: List<Regla>): List<String> =
  reglas.mapNotNull { regla -> regla(pedido) }

fun main() {
  println(validar(Pedido("p1", "AR", 100.0), reglas))
  println(validar(Pedido("", "BR", -5.0), reglas))
}
```

Salida:

```
[]
[El total debe ser positivo, No enviamos a BR, Falta el id]
```

Agregar una regla nueva es agregar una lambda a la lista: `validar` no cambia. Las funciones son **datos**: se guardan en listas, se pasan y se recorren como cualquier otro valor.

</details>

-----

## Checkpoint

Responde sin mirar las lecciones. Si dudas en alguna, vuelve a la lección indicada.

<details>
<summary>1. ¿Qué tipo tiene <code>::mayorQue</code> y qué tipo tiene <code>mayorQue(100.0)</code>?</summary>

`::mayorQue` es `(Double) -> (Pedido) -> Boolean`: la función sin llamar. `mayorQue(100.0)` es `(Pedido) -> Boolean`: lo que la función devuelve. Ver [Funciones de orden superior](01-funciones-de-orden-superior.md).

</details>

<details>
<summary>2. ¿Qué es una closure y qué captura la lambda de <code>crearContador</code>?</summary>

Una lambda que recuerda las variables del lugar donde se creó, aunque esa función ya haya terminado. La de `crearContador` captura `var cuenta`, y cada llamada a `crearContador()` crea una `cuenta` nueva.

</details>

<details>
<summary>3. ¿Por qué <code>pedidos.first { it.pais == "BR" }</code> puede romper tu programa y qué usarías?</summary>

Si no hay coincidencia, `first` lanza `NoSuchElementException`. Usa `firstOrNull` (o `find`) y maneja el `null` con `?.` y `?:`.

</details>

<details>
<summary>4. ¿Por qué <code>"  x  ".apply { trim() }</code> no limpia el texto?</summary>

`apply` devuelve el objeto original, y `trim()` no modifica el `String`: devuelve uno nuevo que se pierde. Ver [Funciones de alcance](02-funciones-de-alcance.md).

</details>

<details>
<summary>5. ¿Cuándo un <code>return</code> dentro de una lambda sale de la función de afuera?</summary>

Cuando la lambda se pasa a una función `inline` (como `forEach`): se copia dentro de la función que llama, y `return` sale de ella. Para salir solo de la lambda, `return@forEach`. Ver [`inline`](03-inline.md).

</details>

<details>
<summary>6. ¿Cuándo dejarías los callbacks por un resultado <code>sealed</code> o por corrutinas?</summary>

Cuando los pasos dependen unos de otros y los callbacks empiezan a anidarse (callback hell). Los callbacks siguen sirviendo para eventos y para funciones con uno o dos finales. Ver [Callbacks](04-callbacks.md).

</details>

-----

Vuelve al índice de la carpeta: [README](README.md).
