# Funciones de orden superior y lambdas

## En una frase

Una función de orden superior **recibe o devuelve otra función**; con ellas separas el "qué hacer" del "cómo recorrer o cuándo ejecutar", y son la base de las operaciones sobre colecciones y de los DSL de Kotlin.

-----

## Antes de empezar

Conviene que ya sepas:

* Qué es una lambda y un tipo de función como `(Int, Int) -> Int`: [Funciones](../05-funciones/01-funciones.md).
* Qué es una extension function y su receptor: [Extension functions](../08-poo-avanzada/03-extension-functions.md).

Palabras nuevas (también están en el [Glosario](Glosario.md)):

* **Función de orden superior:** Función que recibe otra función como parámetro o la devuelve.
* **Tipo de función:** La "forma" de una función: `(String) -> Boolean` recibe un `String` y devuelve un `Boolean`.
* **Trailing lambda:** Escribir la última lambda fuera de los paréntesis.
* **`it`:** Nombre implícito del parámetro cuando la lambda tiene uno solo.
* **Referencia a función (`::`):** Usar una función existente como valor, sin llamarla.
* **Lambda con receptor:** Lambda en la que `this` es un objeto de un tipo dado: `Html.() -> Unit`.
* **Closure:** Una lambda que "recuerda" las variables del lugar donde se creó.

-----

## El problema: lógica repetida con un pedacito distinto

```kotlin
fun pedidosGrandes(pedidos: List<Pedido>): List<Pedido> {
  val resultado = mutableListOf<Pedido>()
  for (p in pedidos) if (p.total > 1000) resultado.add(p)
  return resultado
}

fun pedidosDeArgentina(pedidos: List<Pedido>): List<Pedido> {
  val resultado = mutableListOf<Pedido>()
  for (p in pedidos) if (p.pais == "AR") resultado.add(p)
  return resultado
}
```

Las dos funciones son idénticas salvo **la condición**. ¿Y si la condición fuera un parámetro?

## Recibir una función como parámetro

```kotlin
fun filtrar(pedidos: List<Pedido>, condicion: (Pedido) -> Boolean): List<Pedido> {
  val resultado = mutableListOf<Pedido>()
  for (p in pedidos) if (condicion(p)) resultado.add(p)
  return resultado
}

val grandes = filtrar(pedidos, { p -> p.total > 1000 })
val argentinos = filtrar(pedidos, { p -> p.pais == "AR" })
```

`condicion` es de tipo `(Pedido) -> Boolean`: cualquier función que reciba un `Pedido` y devuelva un `Boolean`. El bucle se escribe **una vez**; lo que cambia entra como parámetro.

## Tres atajos de sintaxis

**1. Trailing lambda:** si el último parámetro es una función, la lambda va fuera de los paréntesis.

```kotlin
filtrar(pedidos) { p -> p.total > 1000 }
```

**2. `it`:** si la lambda tiene **un solo** parámetro, no hace falta nombrarlo.

```kotlin
filtrar(pedidos) { it.total > 1000 }
```

**3. Sin paréntesis:** si la lambda es el **único** parámetro, los paréntesis desaparecen.

```kotlin
fun repetir(accion: () -> Unit) { accion(); accion() }

repetir { println("hola") }
```

Esto explica por qué `routing { ... }` o `transaction { ... }` parecen palabras del lenguaje: son funciones comunes cuyo único parámetro es una lambda.

## Las operaciones de colecciones

La librería estándar ya trae las funciones de orden superior que necesitas. Casi nunca hace falta un bucle a mano:

```kotlin
val pedidos = listOf(
  Pedido("p1", "AR", 1500.0),
  Pedido("p2", "UY", 300.0),
  Pedido("p3", "AR", 800.0),
)

pedidos.filter { it.pais == "AR" }            // [p1, p3]
pedidos.map { it.id }                         // ["p1", "p2", "p3"]
pedidos.sumOf { it.total }                    // 2600.0
pedidos.any { it.total > 1000 }               // true
pedidos.all { it.total > 0 }                  // true
pedidos.firstOrNull { it.pais == "CL" }       // null
pedidos.groupBy { it.pais }                   // {AR=[p1, p3], UY=[p2]}
pedidos.associateBy { it.id }                 // {p1=..., p2=..., p3=...}
pedidos.sortedByDescending { it.total }       // [p1, p3, p2]
pedidos.partition { it.total > 1000 }         // ([p1], [p2, p3])
```

Y se encadenan, leyéndose como una receta:

```kotlin
val totalArgentina = pedidos
  .filter { it.pais == "AR" }
  .sumOf { it.total }
```

`fold` es la más general: recorre la lista acumulando un resultado.

```kotlin
val resumen = pedidos.fold(0.0) { acumulado, pedido -> acumulado + pedido.total }
```

Con colecciones **muy** grandes, cada paso de la cadena crea una lista intermedia. `asSequence()` procesa elemento por elemento, sin listas intermedias:

```kotlin
val primerosCaros = pedidos.asSequence()
  .filter { it.total > 1000 }
  .map { it.id }
  .take(10)
  .toList()
```

## Referencias a funciones

Si la lambda solo llama a una función existente, puedes pasar la función directamente con `::`:

```kotlin
fun esValido(p: Pedido) = p.total > 0

pedidos.filter(::esValido)     // en vez de filter { esValido(it) }
pedidos.map(Pedido::id)        // en vez de map { it.id }
ids.map(::PedidoId)            // un constructor también es una función
```

## Devolver una función

```kotlin
fun mayorQue(minimo: Double): (Pedido) -> Boolean = { it.total > minimo }

val esGrande = mayorQue(1000.0)
pedidos.filter(esGrande)
```

La lambda que se devuelve **recuerda** `minimo` aunque `mayorQue` ya terminó. Eso es una **closure**: la lambda captura las variables del lugar donde se creó.

## Lambdas con receptor: la base de los DSL

En la lección de extension functions viste `fun String.esEmail()`, donde `this` es el `String`. Una **lambda con receptor** es lo mismo, pero como lambda:

```kotlin
val saludar: String.() -> String = { "Hola, ${this.uppercase()}" }

"ana".saludar() // "Hola, ANA"
```

El tipo `String.() -> String` significa: "una función que se ejecuta **sobre** un `String` (que es `this`) y devuelve un `String`".

¿Para qué sirve? Para construir **DSL**: bloques donde puedes llamar funciones de un objeto sin nombrarlo.

```kotlin
class Menu {
  val items = mutableListOf<String>()
  fun item(nombre: String) { items.add(nombre) }
}

fun menu(construir: Menu.() -> Unit): Menu {
  val m = Menu()
  m.construir()   // ejecuta el bloque con `m` como `this`
  return m
}

val principal = menu {
  item("Inicio")      // this.item("Inicio")
  item("Pedidos")
}
```

Así funcionan los bloques que ya usaste:

| Lo que escribes | Lo que es |
| --- | --- |
| `routing { get("/") { ... } }` | `routing` recibe `Routing.() -> Unit`; `get` es una función de `Route` |
| `module { single { ... } }` | `module` recibe `Module.() -> Unit` |
| `PedidosTable.insert { it[id] = ... }` | Lambda que recibe la sentencia como parámetro `it` |
| `HikariConfig().apply { jdbcUrl = ... }` | `apply` ejecuta el bloque con la config como `this` (siguiente lección) |

-----

## Resumen en 5 líneas

1. Una función de orden superior recibe o devuelve funciones; separa el recorrido o el control de la lógica que cambia.
2. Trailing lambda, `it` y la ausencia de paréntesis hacen que `transaction { }` parezca parte del lenguaje.
3. `filter`, `map`, `sumOf`, `groupBy`, `associateBy`, `any`, `fold` reemplazan casi todos los bucles; `asSequence()` evita listas intermedias.
4. `::funcion` pasa una función existente como valor; una lambda devuelta captura sus variables (closure).
5. Una lambda con receptor (`T.() -> Unit`) se ejecuta con `this` = `T`, y es la base de los DSL de Ktor, Koin y Exposed.

-----

## Siguiente lección

[2. Funciones de alcance](02-funciones-de-alcance.md): `let`, `run`, `with`, `apply` y `also`.
