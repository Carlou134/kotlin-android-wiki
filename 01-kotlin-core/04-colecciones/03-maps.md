# Maps

## Introduction to Maps

La colección final que cubriremos en esta sección es un **mapa** (map). Un mapa consiste en un grupo de pares clave-valor donde cada clave corresponde a un único valor. Un par clave-valor se denomina *entrada* (entry) y se asemeja a dos piezas de datos conectadas, donde una clave es un identificador único y el valor representa su dato correspondiente.

Los mapas son aplicables en una variedad de escenarios de programación y su sintaxis se asemeja a datos reales de la web. Observa la imagen de la derecha. La leyenda tiene la forma de un mapa de Kotlin y proporciona información sobre el tipo de animales que el viajero debería esperar encontrar y a los que debe prestar atención en su camino.

El nombre de cada lugar tiene la forma de una **clave** (key), y cada animal correspondiente es el **valor** (value). Observa cómo los datos en un mapa de Kotlin se encierran entre llaves (`{}`) en lugar de corchetes (`[]`), como en las listas y los conjuntos.

En esta lección, exploraremos otras características únicas de la colección de mapas, así como también cómo declarar un mapa y utilizar sus diversas operaciones integradas.

-----

## Creating an Immutable Map

Un mapa inmutable contiene entradas de solo lectura que no pueden modificarse. Los mapas inmutables se utilizan más comúnmente cuando todos los datos necesarios ya existen en el grupo y no se necesita ninguna modificación adicional.

Para declarar un mapa inmutable, necesitamos usar la palabra clave `mapOf` seguida de un par de paréntesis:

```kotlin
val/var nombreMapa = mapOf(clave1 to valor1, clave2 to valor2, clave3 to valor3)
```

- Dentro de los paréntesis deben existir claves únicas junto con sus valores correspondientes.
- La palabra clave `to` se utiliza para vincular cada clave con su valor.
- Cada entrada debe estar separada por una coma.

**Nota:** Las claves dentro de un mapa deben ser únicas; sin embargo, los valores pueden duplicarse.

Para aplicar lo que hemos aprendido hasta ahora, creemos un mapa de instrumentos:

```kotlin
var instruments = mapOf("String" to "Cello", "Brass" to "Saxophone", "Woodwinds" to "Flute")
```

Imprimir el valor de `instruments` dará como resultado la siguiente salida:

```kotlin
println(instruments) // {String=Cello, Brass=Saxophone, Woodwinds=Flute}
```

Observa cómo:

- Cada clave está vinculada a su valor con un `=` en lugar de la palabra clave `to` utilizada en la declaración original.
- El mapa está envuelto en llaves (`{}`) en lugar de corchetes (`[]`) o paréntesis.

-----

## Retrieving Keys and Values

Como hemos visto con otros tipos de colecciones en Kotlin, podemos recuperar y acceder al contenido de una colección con la ayuda de funciones integradas y otras herramientas. Con la colección de mapas, podemos recuperar un valor usando la sintaxis abreviada `[clave]`.

Supongamos el siguiente ejemplo que contiene nombres de bandas famosas como claves y cantantes principales como valores:

```kotlin
var leadSingers = mapOf("The Rolling Stones" to "Mick Jagger", "Blondie" to "Debbie Harry", "Queen" to "Freddie Mercury")
```

Podemos acceder a cualquier valor envolviendo su clave entre corchetes:

```kotlin
leadSingers["Blondie"] // Debbie Harry
```

Si estamos construyendo una aplicación que mostrará los nombres de las bandas en la parte superior de la página, podemos recopilar todas las claves a la vez usando la propiedad `keys`:

```kotlin
leadSingers.keys // [The Rolling Stones, Blondie, Queen]
```

De manera similar, se puede devolver un conjunto de todos los valores del mapa usando la propiedad `values`:

```kotlin
leadSingers.values // [Mick Jagger, Debbie Harry, Freddie Mercury]
```

Cada expresión se puede envolver en una instrucción `println()` para ver su salida o almacenarse en una variable para su uso posterior.

-----

## Creating a Mutable Map

Como hemos visto con las listas y los conjuntos, un mapa se puede declarar como mutable o inmutable. Hasta ahora, solo hemos trabajado con mapas cuyas entradas se utilizan para operaciones de solo lectura. Exploremos cómo declarar mapas mutables y tener más control sobre los datos que contienen.

Para declarar un mapa mutable, usa el término `mutableMapOf` en una declaración de mapa:

```kotlin
var/val nombreMapa = mutableMapOf(clave1 to valor1, clave2 to valor2, clave3 to valor3)
```

La sintaxis anterior se puede usar para declarar el siguiente mapa mutable de estudiantes y sus edades:

```kotlin
var students = mutableMapOf("Willow" to 15, "Elijah" to 17, "Florence" to 16, "Muhammed" to 15)
```

Usando la sintaxis `[clave]`, podemos recuperar y reasignar valores de las claves. Supongamos que Willow acaba de celebrar su cumpleaños y cumplió 16 años. Podemos actualizar el valor de su edad así:

```kotlin
students["Willow"] = 16
```

Observa cómo hemos utilizado el operador de asignación (`=`) para asignar un nuevo valor a la clave `"Willow"`.

**Nota:** En un mapa mutable, los valores se pueden actualizar, pero las claves nunca pueden cambiar. Para actualizar una clave, el par debe eliminarse por completo y volverse a agregar. En el próximo ejercicio, veremos cómo agregar y eliminar valores de un mapa.

----

## Adding and Removing Entries

Dado que un mapa mutable admite operaciones de lectura y escritura, podemos manipular su contenido agregando o eliminando pares clave-valor.

Supongamos que tenemos el siguiente mapa mutable que almacena el árbol nacional de cada país:

```kotlin
var nationalTrees = mutableMapOf("Italy" to "Strawberry Tree", "Greece" to "Olive", "Romania" to "Oak", "Canada" to "Maple")
```

Podemos utilizar la función integrada `put()` de Kotlin para agregar una nueva entrada al mapa mutable:

```kotlin
nationalTrees.put("Albania", "Olive")
```

Dentro de los paréntesis de la función `put()`, hemos colocado la clave seguida de una coma y el valor. El par clave-valor se agregará al final del mapa.

Supongamos que para nuestro programa solo necesitamos rastrear árboles nacionales en Europa. Para eliminar `"Canada"` del mapa, podemos usar la función `remove()`:

```kotlin
nationalTrees.remove("Canada")
```

Observa que solo necesitamos especificar el identificador único, o clave, del par en lugar de la entrada completa.

Al mostrar el mapa final con una instrucción `println()` se obtendrá:

```
{Italy=Strawberry Tree, Greece=Olive, Romania=Oak, Albania=Olive}
```

Explora otras operaciones específicas de mapas aquí, así como otras funcionalidades generales compatibles con mapas en la documentación de Kotlin.

-----

## Review

¡Woohoo! Excelente trabajo. Has completado la lección final sobre los tipos de colecciones en Kotlin. Repasemos lo que hemos visto:

- Un **mapa** es una colección que almacena pares de datos clave-valor.
- Las **claves** dentro de un mapa deben ser únicas; sin embargo, los **valores** pueden duplicarse.
- Un **mapa inmutable** se declara con la palabra clave `mapOf` e indica un mapa cuyas entradas no pueden cambiar a lo largo de un programa.
- Un **mapa mutable** se declara con la palabra clave `mutableMapOf` e indica un mapa cuyas entradas pueden modificarse.
- Los valores dentro de un mapa se pueden acceder usando la sintaxis abreviada `[clave]`.
- Para devolver todos los valores de un mapa, usa la propiedad `values`. Para devolver todas las claves de un mapa, usa la propiedad `keys`.
- La función `put()` acepta una clave y un valor para agregar a un mapa.
- La función `remove()` acepta una clave y elimina la entrada asociada a esa clave.

Ahora has cubierto la tercera y última colección importante que existe en Kotlin. Siéntete libre de utilizar el archivo vacío `Review.kt` y la terminal de salida de la derecha para perfeccionar tu comprensión de los mapas y practicar la escritura de código en Kotlin.

-----