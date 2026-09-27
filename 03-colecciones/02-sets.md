# Sets

## En una frase

Un set es una colección de elementos únicos: si agregas un valor repetido, se descarta automáticamente.

-----

## Antes de empezar

Conviene que ya sepas:

* Sets y listas comparten casi todas sus funciones: [Listas](01-listas.md).

Palabras nuevas (también están en el [Glosario](Glosario.md)):

* **`elementAt()`:** Función que devuelve el elemento de un set en una posición. `elementAtOrNull()` devuelve `null` si esa posición no existe.
* **Set (conjunto):** Colección de elementos únicos: no admite duplicados.

-----

## Introduction to Sets

En esta lección, aprenderemos otra colección fundamental de Kotlin conocida como **conjunto** (set). Un conjunto es una colección **no ordenada** de elementos **únicos**. Es más comúnmente aplicable en programas que trabajan con conjuntos de datos que contienen valores sin duplicados, como nombres de usuario o contraseñas. Al crear o agregar elementos a un conjunto, este filtra automáticamente cualquier duplicado y siempre devuelve una colección única.

A medida que avances en esta lección, notarás que las listas y los conjuntos son muy similares en sintaxis y comparten una gran cantidad de funciones integradas. Su diferencia clave radica en su propósito y comportamiento en un programa. Un desarrollador que construye un programa para las paradas de un metro puede optar por trabajar con una lista, ya que los trenes a menudo se cruzan, mientras que un programa que almacena información de cuentas bancarias probablemente utilizaría un conjunto.

A medida que sigamos explorando otros tipos de colecciones en Kotlin, ten en cuenta las similitudes y diferencias entre cada una de las que has aprendido. Un buen desarrollador no pierde tiempo memorizando cada sintaxis de un libro, sino que, en cambio, se enfoca en comprender qué herramientas están disponibles y cómo se pueden aplicar a un programa.

-----

## Creating an Immutable Set

Como se mencionó anteriormente, las colecciones de Kotlin se dividen en dos categorías: mutables e inmutables, y los conjuntos (sets) no son una excepción. Un conjunto inmutable se declara usando la palabra clave `setOf` e indica un conjunto cuyos valores no pueden cambiar a lo largo de un programa:

```kotlin
var/val nombreConjunto = setOf(valor1, valor2, valor3)
```

**Nota:** Al igual que con las listas, el tipo de conjunto puede ser inferido por el compilador o declarado explícitamente en el código. Por ejemplo, un conjunto compuesto por valores `String` se inferiría como `Set<String>`.

En el código a continuación, estamos creando un nuevo conjunto y almacenándolo en la variable `colorsOfTheRainbow`:

```kotlin
var colorsOfTheRainbow = setOf("red", "orange", "yellow", "green", "blue", "red")
```

Al mostrar `colorsOfTheRainbow` en la consola, observa que hay un color menos en el conjunto. El último elemento, `"red"`, era un `String` duplicado que el conjunto reconoció y eliminó:

```kotlin
println(colorsOfTheRainbow)
// Imprime: [red, orange, yellow, green, blue]
```

🌈

----

## Accessing Elements in a Set

A diferencia de las listas, un conjunto (set) es una colección no ordenada, por lo tanto no podemos usar corchetes e índices para recuperar sus elementos. Sin embargo, un conjunto es una herramienta inteligente que almacena sus elementos en el orden en que cada elemento fue insertado. Podemos utilizar este orden junto con una función, `elementAt()`, para acceder y recuperar elementos.

La función `elementAt()` acepta un valor entero y devuelve el elemento en esa posición. Esta función es útil para colecciones que no poseen acceso por índice.

```kotlin
nombreLista.elementAt(entero)
```

Supongamos que estamos escribiendo un programa que lleva un registro de los animales que ya no están en peligro de extinción:

```kotlin
var nonEndangered = setOf("Louisiana Black Bear", "Northern Brown Kiwi", "Gray Wolf", "Arabian Oryx")
```

Para recuperar el animal `"Gray Wolf"`, usaríamos la siguiente sintaxis:

```kotlin
nonEndangered.elementAt(2) // Gray Wolf
```

Dado que Kotlin es un lenguaje con seguridad de nulos (null safe), anima a los desarrolladores a evitar errores de `NullPointerException` siempre que sea posible utilizando variantes más seguras de las funciones.

En caso de que un elemento no exista en una posición especificada, nuestro código arrojaría un error:

```kotlin
nonEndangered.elementAt(5) // Exception error
```

**Mensaje de error:**

```
Exception in thread "main" java.lang.IndexOutOfBoundsException: Collection doesn't contain element at index 5.
```

Para evitar esto, podemos usar la función `elementAtOrNull()`, una variante más segura de `elementAt()` que devuelve `null` en lugar de un error cuando un elemento no existe en el conjunto:

```kotlin
nonEndangered.elementAtOrNull(5) // null
```

---

## Creating a Mutable Set

Con los conjuntos (sets) inmutables, no tenemos mucho control sobre el contenido que almacenan, pero con los conjuntos mutables podemos agregar, eliminar o reemplazar tantos elementos como queramos. Esto nos permite trabajar tanto con conjuntos que están poblados con valores como con conjuntos vacíos, que son conjuntos que se inicializan sin valores pero que pueden poblarse más adelante.

Para inicializar un conjunto mutable vacío, usaríamos la siguiente sintaxis:

```kotlin
var/val nombreConjunto = mutableSetOf<Tipo>()
```

- El término `mutableSetOf` se usa para declarar un conjunto mutable.
- El término va seguido del tipo de dato que esperamos que contenga el conjunto y un par de paréntesis vacíos.

Supongamos que estamos desarrollando un programa para el administrador de un edificio que quiere llevar un registro de los apartamentos que posee. Desafortunadamente, aún no hemos recibido la información de los apartamentos, así que creamos un conjunto mutable y esperamos hasta poder poblarlo con valores:

```kotlin
var totalApts = mutableSetOf<String>()
```

Mientras esperamos, practiquemos declarar un conjunto que viene previamente poblado con valores. La sintaxis es la siguiente:

```kotlin
var/val nombreConjunto = mutableSetOf(valor1, valor2, valor3)
```

No es necesario especificar el tipo de conjunto aquí, ya que será automáticamente inferido por el compilador. Veamos un ejemplo:

```kotlin
var nycRadioStations = mutableSetOf(106.7, 101.1, 97.1, 103.5)
```

El compilador inferirá `mutableSetOf<Double>` a partir del código anterior y solo aceptará valores de tipo `Double` dentro de este conjunto en adelante.

----

## Adding and Removing Elements

Para agregar o eliminar elementos de un conjunto (set), debemos utilizar las funciones integradas de Kotlin para conjuntos. Afortunadamente, muchas de estas funciones también se pueden aplicar a otros tipos de colecciones, por lo que es posible que reconozcas algunas de estas sintaxis de una lección anterior.

Para agregar un solo elemento a un conjunto, usaríamos la función `add()` como hicimos en la lección de listas; sin embargo, si estuviéramos interesados en agregar muchos elementos a la vez a un conjunto, la función `addAll()` sería más apropiada. Ten en cuenta que, a diferencia de las listas, no podemos agregar un elemento que ya existe en el conjunto.

Veamos cómo funciona `addAll()`. Recordemos el ejemplo del edificio del ejercicio anterior. El administrador nos respondió con la siguiente lista de apartamentos:

```kotlin
var aptList = listOf("1A", "1B", "1C", "2A", "2B", "2C", "2C", "1A")
```

Estamos listos para poblar nuestro conjunto vacío, `totalApts`, con estos valores. Con la ayuda de la función `addAll()`, podemos poblar nuestro conjunto con múltiples valores a la vez:

```kotlin
totalApts.addAll(aptList) 

println(totalApts) // Imprime: [1A, 1B, 1C, 2A, 2B, 2C]
```

Parece que el administrador pudo haber agregado accidentalmente números de apartamento duplicados. Afortunadamente, el conjunto eliminó los elementos duplicados y nos quedamos con datos únicos.

Por el contrario, también podemos eliminar todos los elementos de una colección usando la función `clear()`. Supongamos que el edificio está vendiendo su propiedad y es necesario eliminar todos los apartamentos del conjunto de datos:

```kotlin
totalApts.clear()
println(totalApts) // Imprime: []
```

`addAll()` y `clear()` son formas muy útiles, rápidas y eficientes de agregar o eliminar grandes cantidades de datos de una colección a la vez.

----

## Additional Set Functions

Además de agregar y eliminar elementos de un conjunto (set), la colección de conjuntos admite una gran variedad de otras funcionalidades. Puedes encontrar la lista completa en la documentación de Kotlin.

En este ejercicio, exploraremos algunas de las funciones más comunes de los conjuntos usando el siguiente ejemplo:

```kotlin
var numbers = setOf(10, 15, 20, 25)
```

Para recuperar el **primer** elemento de cualquier conjunto, podemos usar la función `first()`:

```kotlin
numbers.first() // 10
```

Para recuperar el **último** elemento de cualquier conjunto, podemos usar la función `last()`:

```kotlin
numbers.last() // 25
```

Finalmente, si quisiéramos **sumar** todos los elementos de un conjunto, podemos usar la función `sum()`:

```kotlin
numbers.sum() // 70
```

**Nota:** Para ver la salida de cada llamada a una función en un conjunto, debe estar envuelta en una instrucción `println()`.

-----

## Review

¡Excelente trabajo! En esta lección, hemos aprendido los siguientes conceptos:

- Un **conjunto (set)** es una colección desordenada de Kotlin que almacena elementos únicos.
- Un **conjunto inmutable** se declara con la palabra clave `setOf` e indica un conjunto cuyos valores no pueden cambiar a lo largo de un programa.
- Un **conjunto mutable** se declara con la palabra clave `mutableSetOf` e indica un conjunto cuyos valores pueden modificarse.
- Los elementos dentro de un conjunto se pueden acceder usando las funciones `elementAt()` o `elementAtOrNull()`.
- Las funciones `addAll()` y `add()` se utilizan para agregar elementos a una colección.
- Las funciones `first()` y `last()` devuelven el primer y el último elemento de una colección.
- La función `clear()` elimina todos los elementos de una colección.

Ahora has cubierto la segunda colección importante que existe en Kotlin, así como en muchos otros lenguajes de programación. Siéntete libre de utilizar el archivo vacío `Review.kt` y la terminal de salida de la derecha para perfeccionar tu comprensión de los conjuntos y practicar la escritura de código en Kotlin.

-----

## Resumen en 5 líneas

1. Un set guarda elementos únicos: los duplicados se descartan solos.
2. `setOf(...)` crea uno de solo lectura; `mutableSetOf(...)` uno modificable.
3. No se accede con `[índice]`: se usa `elementAt()` o `elementAtOrNull()`.
4. `add()` y `addAll()` agregan elementos; `clear()` vacía la colección.
5. `first()`, `last()` y `sum()` también funcionan en sets. Úsalo cuando importa que no haya repetidos (usuarios, emails).

-----

## Siguiente lección

Sigue con: [Maps](03-maps.md).
