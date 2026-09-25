# Lists

## Introduction to Lists

En esta sección, vamos a explorar los diversos tipos de colecciones que existen en Kotlin. Una **colección** es un grupo de elementos que existen en una sola estructura. El primer tipo de colección que aprenderemos es uno de los más utilizados en Kotlin y en muchos otros lenguajes de programación: una **lista**.

Una lista en Kotlin es muy similar a una lista del mundo real: representa una colección ordenada de elementos que puede contener duplicados. Estos elementos pueden ser de cualquier tipo de dato primitivo y representan una gran variedad de situaciones reales. Estos pueden incluir calificaciones de estudiantes, valoraciones de restaurantes, una lista de tareas pendientes, y más.

Observa la imagen de la derecha. Después de un largo día navegando por el arroyo, Codey va a montar el campamento y descansar durante la noche. Siendo el desarrollador de Kotlin respetuoso con el medio ambiente que es, ha optado por hacer una lista virtual de tareas que debe cumplir antes del anochecer:

```kotlin
var toDo = listOf("gather wood", "start a fire", "set up tent", "eat dinner", "stargaze")
```

Imprimir la lista daría como resultado la siguiente salida:

```
[gather wood, start a fire, set up tent, eat dinner, stargaze]
```

Observa cómo la sintaxis de una lista en su definición es diferente de su salida. Si Kotlin no es tu primer lenguaje de programación, es posible que reconozcas que la sintaxis de salida con corchetes (`[]`) es idéntica a la de un **array** (arreglo).

A diferencia de las listas, los arreglos en Kotlin tienen un tamaño fijo, por lo que no son la estructura de datos más conveniente ni fácilmente aplicable. Por esa razón, nos centraremos en explorar las listas, pero te animamos a que leas sobre los arreglos en la documentación de Kotlin.

---

## Arrays y listas mutables en Kotlin, explicado simple

Las colecciones sirven para guardar varios datos dentro de una misma estructura.

En Kotlin vas a usar mucho dos tipos:

- **arrays**, cuando tenés una cantidad fija de elementos;
- **listas mutables**, cuando necesitás agregar, cambiar o eliminar elementos.

### Crear un array

Un array guarda varios valores del mismo tipo.

```kotlin
val emails = arrayOf("Juan@empresa.com", "Carlos@empresa.com", "Wilson@empresa.com")
```

Este array tiene tres elementos. Como los índices empiezan en `0`, el primer elemento está en la posición `0`.

```kotlin
println(emails[0]) // Juan@empresa.com
```

### Modificar un elemento del array

Podés cambiar un valor usando su índice:

```kotlin
emails[0] = "nuevoCorreoJuan@empresa.com"
```

También podés usar `set()`:

```kotlin
emails.set(0, "otroCorreoJuan@empresa.com")
```

Ojo: un array tiene tamaño fijo. Podés cambiar sus valores, pero no agregar ni quitar posiciones.

### Cuidado con índices fuera de rango

Si el array tiene tres elementos, sus índices son `0`, `1` y `2`.

```kotlin
println(emails[3]) // Error
```

Esto falla porque la posición `3` no existe. No es un detalle menor: acceder mal a un índice rompe el programa.

### Crear una lista mutable

Una lista mutable sí puede crecer o reducirse.

```kotlin
val listaEmails = mutableListOf<String>()
```

Podés agregar elementos con `add()`:

```kotlin
listaEmails.add("nuevoEmail@empresa.com")
```

Y también agregar varios elementos de golpe:

```kotlin
listaEmails.addAll(arrayOf("Juan@empresa.com", "Carlos@empresa.com"))
```

### Remover o insertar elementos

Para eliminar un elemento específico:

```kotlin
listaEmails.remove("Juan@empresa.com")
```

Para insertar un elemento en una posición concreta:

```kotlin
listaEmails.add(0, "Juan@empresa.com")
```

Eso coloca el email en la primera posición.

### Mostrar una colección ordenadamente

Para mostrar todos los elementos como texto, usá `joinToString()`:

```kotlin
println("Lista completa: ${listaEmails.joinToString()}")
```

Esto imprime los elementos separados por comas, de forma más clara que mostrar la colección sin formato.

**Idea clave:** usá arrays cuando el tamaño no va a cambiar. Usá listas mutables cuando necesitás agregar, quitar o reorganizar datos.

---

## `listOf`, `mutableListOf` y arrays: diferencias importantes

Cuando trabajás con colecciones en Kotlin, elegir bien la estructura importa.

No es lo mismo usar:

- `arrayOf`;
- `listOf`;
- `mutableListOf`.

Cada una tiene reglas distintas para leer, modificar, agregar o eliminar datos.

### Lista inmutable con `listOf`

Una lista creada con `listOf` es una lista de solo lectura.

```kotlin
val emails = listOf("juan@empresa.com", "ana@empresa.com", "wilson@empresa.com")
```

Podés leer sus elementos:

```kotlin
println(emails[0]) // juan@empresa.com
```

Pero no podés modificar, agregar ni eliminar elementos directamente:

```kotlin
// emails[0] = "nuevo@empresa.com" // Error
// emails.add("otro@empresa.com")  // Error
```

Esto es importante: `listOf` no es para cambiar datos, es para trabajar con una colección que no debería modificarse.

### Funciones útiles en listas

Las listas tienen funciones muy cómodas para acceder a datos sin calcular índices manualmente.

```kotlin
val listOfEmails = listOf("juan@empresa.com", "ana@empresa.com", "wilson@empresa.com")

println(listOfEmails.first())     // juan@empresa.com
println(listOfEmails.last())      // wilson@empresa.com
println(listOfEmails.lastIndex)   // 2
```

- `first()` devuelve el primer elemento.
- `last()` devuelve el último elemento.
- `lastIndex` devuelve el índice del último elemento.

### Extraer una parte con `subList`

`subList()` permite obtener una parte de una lista.

```kotlin
val listOfEmails = listOf("juan@empresa.com", "ana@empresa.com", "wilson@empresa.com")
val nuevaLista = listOfEmails.subList(1, 3)

println(nuevaLista) // [ana@empresa.com, wilson@empresa.com]
```

El primer índice se incluye. El segundo índice no se incluye.

Por eso `subList(1, 3)` toma las posiciones `1` y `2`, pero no intenta tomar la posición `3`.

### String templates: `$variable` vs `${expresion}`

Para imprimir una variable simple, alcanza con `$`.

```kotlin
val nombre = "Ana"
println("Hola, $nombre")
```

Usá `${}` cuando necesitás evaluar una expresión más compleja.

```kotlin
println("Cantidad de emails: ${listOfEmails.size}")
```

### Lista mutable con `mutableListOf`

Una lista mutable permite cambiar su contenido y también su tamaño.

```kotlin
val asuntos = mutableListOf(
  "Reunión de equipo",
  "Reporte mensual",
  "Invitación de evento"
)

println("Inicial: $asuntos")

asuntos[1] = "Reporte semanal"
println("Modificada: $asuntos")

asuntos.removeAt(0)
println("Final: $asuntos, tamaño: ${asuntos.size}")
```

Resultado final:

```text
[Reporte semanal, Invitación de evento]
```

Acá usamos `mutableListOf` porque la lista cambia durante la ejecución: primero modificamos un elemento y después eliminamos otro.

### ¿Cuál conviene usar?

- Usá `arrayOf` cuando el tamaño es fijo y querés una estructura simple.
- Usá `listOf` cuando querés leer datos sin modificarlos.
- Usá `mutableListOf` cuando necesitás agregar, eliminar o cambiar elementos.

**Idea clave:** no elijas una colección por costumbre. Elegila según la operación que necesitás. Ahí empieza el código pensado, no el código escrito en automático.

---


## Creating an Immutable List

En Kotlin, los términos "mutable" e "inmutable" se aplican no solo a las variables sino también a las colecciones. Al igual que una variable inmutable representa un valor de solo lectura, una lista inmutable contiene elementos que nunca deben modificarse. Su sintaxis es la siguiente:

```kotlin
var/val nombreLista = listOf(valor1, valor2, valor3)
```

- Para que una lista sea utilizada y referenciada a lo largo de un programa, debe almacenarse en una variable. Asegúrate de usar las palabras clave `val` o `var` en tu declaración, así como un nombre descriptivo para la lista.
- La creación de la lista Kotlin comienza en el lado derecho del operador de asignación con el término `listOf`. Esto indica una lista inmutable.
- Luego, el término va seguido de un par de paréntesis que contienen una colección de elementos separados por comas.
- Al igual que con las declaraciones de variables, el tipo de lista puede ser inferido por el compilador o declarado por el usuario. Por ejemplo, una lista de valores `String` se referenciaría como `List<String>`.
- **Nota:** Una vez inicializada como `List<String>`, la lista solo puede contener valores `String`.

Teniendo en cuenta la sintaxis anterior, creemos una lista inmutable de valores `String` que representen los países que hemos visitado:

```kotlin
var countriesVisited = listOf("Japan", "Colombia", "Kenya", "Jordan")
```

Dado que `countriesVisited` se declaró como una variable mutable con la palabra clave `var`, podemos reasignar el valor de `countriesVisited` para almacenar una lista diferente si así lo deseamos. Sin embargo, dado que la lista se declaró inmutable con `listOf`, el contenido de la lista no puede cambiar. Siempre deben seguir siendo `"Japan"`, `"Colombia"`, `"Kenya"` y `"Jordan"`.

```kotlin
println(countriesVisited) 
// Imprime: [Japan, Colombia, Kenya, Jordan]
```

Por último, cuando se imprime en pantalla, la lista se muestra dentro de corchetes (`[]`) en lugar de paréntesis.

Hemos aprendido varias reglas y conceptos nuevos en este ejercicio. No te desanimes si no todo se asimila de inmediato; tendremos mucha práctica a lo largo de esta lección. ¡Sigue adelante y creemos tu primera lista en Kotlin! 📝

----

## Accessing Elements in a List

Una colección no nos sirve de nada si no podemos acceder a su contenido ni trabajar con él. Para acceder a los elementos dentro de una lista, debemos hacer referencia al "índice" del elemento. Un índice representa una posición numérica dentro de una lista.

```kotlin
val oakIslandArtifacts = listOf("Lead cross", "Rhodolite garnet brooch", "Gold brooch")
```

Aquí tienes un desglose de los índices de los elementos de la lista anterior:

| Elemento | Índice |
|----------|--------|
| "Lead cross" | 0 |
| "Rhodolite garnet brooch" | 1 |
| "Gold brooch" | 2 |

Observa cómo el conteo comienza con 0 y no con 1; esto se debe a que Kotlin es un lenguaje basado en cero (zero-based) y el conteo en cada colección de Kotlin comienza en 0.

Teniendo en cuenta estas posiciones indexadas, podemos construir la siguiente sintaxis para acceder a un elemento:

```
nombreLista[índice]
```

El nombre de la lista va seguido de corchetes que contienen el índice del elemento al que necesitamos acceder. Aplicando este pseudocódigo a nuestra lista anterior, podemos acceder a cada elemento de la siguiente manera:

```kotlin
oakIslandArtifacts[0] // Lead cross
oakIslandArtifacts[1] // Rhodolite garnet brooch
oakIslandArtifacts[2] // Gold brooch
```

Cada declaración puede envolverse en una instrucción `println()` para ver su salida o almacenarse en una variable para su uso posterior.

**Nota:** Intentar acceder a un elemento en un índice que no existe provocará un error. Por ejemplo, si hay 2 elementos en una lista y se especifica el índice 5, el compilador arrojará el siguiente error de fuera de rango (Out of Bound):

```
java.lang.ArrayIndexOutOfBoundsException: Index 5 out of bounds for length 2
```

----

## Creating a Mutable List

Los elementos se pueden acceder desde cualquier lista, pero solo con listas mutables podemos cambiar el contenido de nuestra lista agregando, reemplazando o eliminando elementos. Anteriormente solo hemos creado listas inmutables; en este ejercicio, aprenderemos cómo crear una lista mutable y modificar su contenido.

La sintaxis para declarar una lista mutable es la siguiente:

```kotlin
var/val nombreLista = mutableListOf(valor1, valor2, valor3)
```

Una lista mutable se define mediante el término `mutableListOf`, a diferencia de `listOf` como hemos visto anteriormente. Aunque la sintaxis y la estructura de las listas mutables e inmutables son las mismas, el compilador sabe qué operaciones se pueden o no se pueden realizar en la lista según el término con el que se definió.

Supongamos que tenemos una lista que almacena las actividades diarias de un estudiante:

```kotlin
var dailyActivities = mutableListOf("going to school", "attending tennis practice", "reading")
```

Ahora que es verano y no hay escuela, nos gustaría actualizar el primer elemento de nuestra lista para reflejar la nueva temporada. Podemos llevar nuestro conocimiento previo sobre acceso a elementos un paso más allá y asignar un nuevo valor a la posición indexada de un elemento con la siguiente sintaxis:

```kotlin
dailyActivities[0] = "swimming at the beach"
```

El valor que sigue al operador de asignación reemplazará al valor actual en el índice 0. Imprimir el valor de `dailyActivities` dará como resultado la siguiente salida y la lista actualizada:

```
[swimming at the beach, attending tennis practice, reading]
```

-----

## The size Property

En Kotlin, la propiedad `size` determina la cantidad de elementos dentro de una colección. No solo es aplicable a listas, sino también a otras colecciones con las que trabajaremos en las próximas lecciones.

De manera similar a la propiedad `length` para strings, `size` cuenta el número de elementos en una colección y devuelve un valor entero (`Integer`).

Observa la siguiente lista de los principales ríos europeos:

```kotlin
val majorRivers = listOf("Volga", "Danube", "Loire", "Rhine", "Elbe")
```

Podemos agregar `size` a `majorRivers` y obtener el número de elementos en la lista:

```kotlin
println(majorRivers.size) // Imprime: 5
```

Además, este código se puede usar para construir un `String` informativo sobre nuestra lista:

```kotlin
println("There are ${majorRivers.size} major rivers in Europe.")
// Imprime: There are 5 major rivers in Europe.
```

Además, `size` se puede usar junto con el índice de un elemento para recuperar el último elemento de una colección:

```kotlin
println(majorRivers[majorRivers.size - 1]) // Imprime: Elbe
```

- Dentro de los corchetes, `majorRivers.size` equivale a 5. Restar 1 de este valor da como resultado 4, que se convierte en el índice numérico final dentro de los corchetes.
- `majorRivers[4]` devuelve el quinto y último elemento de la lista, `"Elbe"`.

Esta estrategia se usa más comúnmente cuando se itera sobre una colección o simplemente para obtener el último elemento de una lista cuando no sabemos cuántos elementos hay en total. Aprenderemos más sobre cómo iterar sobre una colección más adelante en el curso.

-----

## List Functions

Cada colección en Kotlin contiene una serie de funciones integradas que nos permiten realizar ciertas operaciones para recuperar información sobre la colección o manipular sus valores.

Una función que recupera información sobre una lista solo requeriría acceso de lectura, mientras que una función que cambia el contenido de una lista requeriría tanto acceso de lectura como de escritura. Como resultado de esto, una lista mutable admite funciones que poseen funcionalidades de lectura y escritura, mientras que una lista inmutable admite funciones que poseen operaciones de solo lectura. Analicémoslo más de cerca:

**Funciones para listas mutables e inmutables:**

```kotlin
var vowels = listOf('A', 'E', 'I', 'O', 'U')
```

La función `contains()` acepta un solo valor dentro de sus paréntesis y devuelve un valor booleano (`true` o `false`) dependiendo de si ese valor existe o no dentro de una lista. No necesita cambiar el contenido de la lista, por lo tanto, se puede usar en una lista inmutable.

```kotlin
println(vowels.contains('A')) // Imprime: true
println(vowels.contains('B')) // Imprime: false
```

La función `random()` también solo requiere acceso de solo lectura y devuelve un elemento aleatorio de la lista:

```kotlin
println(vowels.random()) // Imprime: (por ejemplo) I
```

**Funciones SOLO para listas mutables:**

```kotlin
var primeNumbers = mutableListOf(4, 5, 7, 11, 13)
```

La función `add()` acepta un solo valor dentro de sus paréntesis y agrega ese valor al final de una lista:

```kotlin
primeNumbers.add(17) 
println(primeNumbers) // Imprime: [4, 5, 7, 11, 13, 17]
```

Dado que esta función cambia el contenido de la lista, solo se puede usar en listas mutables. De manera similar, la función `remove()`, también usada solo en listas mutables, acepta un solo valor dentro de sus paréntesis y lo elimina de la lista:

```kotlin
primeNumbers.remove(4) 
println(primeNumbers) // Imprime: [5, 7, 11, 13, 17]
```

**Nota:** Si el valor que se desea eliminar no existe, el compilador no arrojará un error y simplemente devolverá `false`.

Estas funciones son solo la punta del iceberg de lo que está disponible para las listas en Kotlin; te animamos a que revises la documentación sobre listas inmutables y listas mutables para explorar otras funcionalidades disponibles.

-----

## Review

¡Excelente trabajo! En esta lección, hemos aprendido los siguientes conceptos:

- Una **lista** es un tipo de Colección de Kotlin que contiene elementos ordenados.
- Una **lista inmutable** se declara con `listOf` e indica una lista cuyos valores no pueden cambiar a lo largo de un programa.
- Una **lista mutable** se declara con `mutableListOf` e indica una lista cuyos valores pueden modificarse.
- Los elementos dentro de una lista se pueden acceder usando su **índice**.
- La propiedad `size` devuelve el número de elementos en una colección.
- Las listas inmutables poseen funcionalidades de solo lectura, mientras que las listas mutables admiten funcionalidades tanto de lectura como de escritura.
- La función `contains()` verifica si un elemento existe en una lista determinada. Devuelve un valor booleano (`true` o `false`).
- La función `add()` agrega un elemento al final de una lista mutable.
- La función `remove()` elimina un elemento de una lista mutable.
- La función `random()` genera un elemento aleatorio a partir de una lista determinada.

Has cubierto un concepto fundamental y importante de programación, utilizado en la mayoría de los programas para almacenar y manipular datos. Siéntete libre de utilizar el archivo vacío `Review.kt` y la terminal de salida de la derecha para perfeccionar tu comprensión de las listas y practicar la escritura de código en Kotlin.

-----
