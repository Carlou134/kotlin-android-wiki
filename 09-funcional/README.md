# Programación funcional

En esta carpeta aprendes a **tratar las funciones como valores**: pasarlas como parámetros, devolverlas, transformar colecciones sin bucles, usar las funciones de alcance (`let`, `run`, `apply`, `also`, `with`) y entender qué hace `inline` y por qué existe. Es la base de todos los DSL que usas en Ktor, Koin y Exposed.

-----

## Antes de empezar

Conviene que ya hayas leído [Funciones](../05-funciones/README.md), sobre todo la parte de literales de función y lambdas, y [Extension functions](../08-poo-avanzada/03-extension-functions.md), porque las lambdas con receptor son su versión "lambda".

Si en algún momento aparece una palabra que no conoces, búscala en el [Glosario](Glosario.md).

-----

## Orden de lectura

Las lecciones se apoyan una en la otra, así que conviene leerlas en este orden:

| Lección | Qué aprendes | Qué necesitas saber antes |
| --- | --- | --- |
| [1. Funciones de orden superior](01-funciones-de-orden-superior.md) | Recibir y devolver funciones, trailing lambdas, `it`, referencias `::`, operaciones de colecciones y lambdas con receptor | Lambdas y extension functions |
| [2. Funciones de alcance](02-funciones-de-alcance.md) | `let`, `run`, `with`, `apply`, `also`, `takeIf`: cuál usar y cuándo no usar ninguna | Lambdas con receptor |
| [3. `inline`](03-inline.md) | El costo de las lambdas, `return` no local, etiquetas, `noinline`, `crossinline` y `reified` | Funciones de orden superior |

-----

## El mapa completo en una mirada

```
Orden superior       ->  "funciones que reciben o devuelven funciones"
   fun reintentar(veces: Int, accion: () -> Unit)
   reintentar(3) { conectar() }                  // trailing lambda
   pedidos.filter { it.total > 100 }.map { it.id }
   pedidos.map(Pedido::id)                       // referencia a función

Lambda con receptor  ->  "un bloque donde `this` es otro objeto"
   fun html(bloque: Html.() -> Unit)              routing { get("/") { ... } }

Funciones de alcance ->  "ejecutar un bloque sobre un objeto"
   x?.let { usar(it) }        HikariConfig().apply { jdbcUrl = "..." }
   pedido.also { log(it) }    with(builder) { ... }      run { calcular() }

inline               ->  "copiar la lambda en el lugar de la llamada"
   lista.forEach { if (it == x) return }         // return sale de la función de afuera
   inline fun <reified T> leer(): T              // T disponible en tiempo de ejecución
```

-----

## Cómo está armada cada lección

1. **En una frase:** qué es, sin jerga.
2. **Antes de empezar:** qué tienes que saber y las palabras nuevas.
3. **El contenido:** la explicación con ejemplos.
4. **Resumen en 5 líneas:** para repasar.
5. **Siguiente lección:** para seguir el orden.

-----

## Recursos

* [Funciones de orden superior y lambdas (documentación oficial)](https://kotlinlang.org/docs/lambdas.html)
* [Funciones de alcance (documentación oficial)](https://kotlinlang.org/docs/scope-functions.html)
* [Funciones inline (documentación oficial)](https://kotlinlang.org/docs/inline-functions.html)

-----

## Después de esta carpeta

Ya entiendes la sintaxis que hay detrás de los DSL de Kotlin. Lo próximo es [Corrutinas](../10-corrutinas/README.md). Vuelve al índice: [README principal](../README.md).
