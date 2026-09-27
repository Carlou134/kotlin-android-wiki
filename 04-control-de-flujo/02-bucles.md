# Loops

## En una frase

Un bucle repite un bloque de código: `for` recorre rangos y colecciones, y `while` repite mientras una condición sea verdadera.

-----

## Antes de empezar

Conviene que ya sepas:

* Condiciones y rangos (`..`): [Condicionales](01-condicionales.md).
* Lo que vas a recorrer: [Listas](../03-colecciones/01-listas.md).
* Recorrer claves y valores: [Maps](../03-colecciones/03-maps.md).

Palabras nuevas (también están en el [Glosario](Glosario.md)):

* **Bucle (loop):** Estructura que repite un bloque de código.
* **Bucle anidado:** Un bucle dentro de otro. El interno se completa entero en cada vuelta del externo.
* **Etiqueta (label):** Nombre terminado en `@` que marca un bucle, para usar `break@nombre` o `continue@nombre` sobre él.
* **Expresión de salto:** `break` (sale del bucle) y `continue` (salta a la siguiente iteración).
* **Iteración:** Cada vuelta que da un bucle.

-----

## Introduction to Loops

¿Por qué escribir 100 líneas de código cuando puedes escribir 2? Cuando nos encontramos con código que queremos repetir, podemos confiar en los **bucles** (loops). Los bucles nos brindan la capacidad de repetir bloques de código específicos tantas veces como queramos.

Digamos que quisiéramos mostrar los elementos de una lista. El proceso no sería tan malo si la lista tuviera 5 elementos. Podríamos escribir cinco instrucciones `println()`. ¿Y si la lista tuviera 100 elementos? ¿O 1,000? ¿O incluso 100,000?

Los bucles responden a la necesidad cuando tenemos que repetir una tarea una y otra vez, incluso si son millones de veces.

En esta lección, veremos bucles que repetirán código un número definido de veces o basándose en una condición específica. Veremos cómo combinar bucles para obtener un mayor número de repeticiones, así como también cómo salir de bucles anticipadamente o saltarnos repeticiones de código.

Al final de esta lección entenderás la mejor manera de repetir tareas de forma controlada mientras mantienes tu código organizado y legible.

----

## Bucles en Kotlin, explicado simple

Un **bucle** permite repetir un bloque de código varias veces.

Sirve para evitar esto:

```kotlin
println("Procesando email 1")
println("Procesando email 2")
println("Procesando email 3")
```

Y escribir algo más flexible:

```kotlin
var emailsPendientes = 3

while (emailsPendientes > 0) {
  println("Procesando email. Quedan $emailsPendientes")
  emailsPendientes -= 1
}
```

La idea es clara: si una tarea se repite, probablemente necesitás un bucle.

### `while`

`while` evalúa una condición antes de ejecutar el bloque.

```kotlin
var emailsPendientes = 3

while (emailsPendientes > 0) {
  println("Procesando email. Quedan $emailsPendientes")
  emailsPendientes -= 1
}
```

Mientras `emailsPendientes > 0` sea `true`, el bloque se repite.

Ojo: si nunca modificás la variable que controla la condición, podés crear un bucle infinito. Eso no es un detalle: ES un bug.

### `do while`

`do while` ejecuta el bloque al menos una vez y después revisa la condición.

```kotlin
var intentos = 0

do {
  intentos += 1
  println("Intento de envío número $intentos")
} while (intentos < 3)
```

Usalo cuando necesitás que el código se ejecute mínimo una vez, por ejemplo para pedir datos o reintentar una operación.

### `for`

`for` es ideal cuando querés recorrer rangos o colecciones.

```kotlin
for (i in 0..5) {
  println(i)
}
```

El rango `0..5` incluye ambos extremos: `0`, `1`, `2`, `3`, `4`, `5`.

También podés recorrer hacia atrás:

```kotlin
for (i in 5 downTo 0) {
  println(i)
}
```

O saltar valores con `step`:

```kotlin
for (i in 0..10 step 2) {
  println(i)
}
```

Esto imprime: `0`, `2`, `4`, `6`, `8`, `10`.

### Recorrer una colección

Cuando tenés una lista, `for` te permite recorrer elemento por elemento sin manejar índices manualmente.

```kotlin
val emails = listOf("juan@test.com", "carlos@test.com", "wilson@test.com")

for (email in emails) {
  println("Correo actual: $email")
}
```

Cada vuelta guarda un elemento distinto dentro de la variable `email`.

### Recorrer una colección con `while`

También se puede hacer con `while`, pero necesitás controlar el índice manualmente.

```kotlin
val emails = listOf("juan@test.com", "carlos@test.com", "wilson@test.com")
var indiceEmail = 0

while (indiceEmail < emails.size) {
  println(emails[indiceEmail])
  indiceEmail += 1
}
```

Funciona, pero para recorrer todos los elementos de una colección, `for` suele ser más claro.

### Ejercicio: contar emails válidos

Este ejemplo lee tres emails, los guarda en una lista mutable y después cuenta cuántos contienen `@`.

```kotlin
val emails = mutableListOf<String>()

for (i in 1..3) {
  print("Email $i: ")
  val entrada = readLine() ?: "email$i@test.com"
  emails.add(entrada)
}

var contador = 0
var posicion = 0

while (posicion < emails.size) {
  if (emails[posicion].contains("@")) {
    contador += 1
  }

  posicion += 1
}

println("Emails válidos: $contador")
```

Acá aparecen varias piezas juntas:

- `mutableListOf<String>()` permite agregar emails con `add()`.
- `for` sirve para pedir tres entradas.
- `while` recorre la lista usando una posición.
- `contains("@")` valida si el texto contiene arroba.

**Idea clave:** usá `for` cuando sabés qué colección o rango querés recorrer. Usá `while` cuando la repetición depende de una condición. Usá `do while` cuando necesitás ejecutar algo al menos una vez.

----

## for Loop

Cuando sabemos cuántas veces queremos que se repita una sección de código, a menudo usamos un bucle `for`.

Un bucle `for` comienza con la palabra clave `for` y va seguido de una declaración que define una variable de bucle seguida de la identificación de un iterador:

```kotlin
for(i in 1..4) {
  println("I am in a loop!")
}
```

- `for` es una palabra clave que se usa para declarar un bucle `for`.
- Definimos `i` como la variable de bucle. Esta variable contiene el valor de la iteración actual y se puede usar dentro del cuerpo del bucle.
- La palabra clave `in` está entre la definición de la variable y el iterador.
- El rango `1..4` es el iterador del bucle `for`.
- Un **iterador** es un objeto que nos permite recorrer y acceder a cada elemento individual en una colección de valores. En la mayoría de los casos, los iteradores que usamos en los bucles `for` son rangos y colecciones. En este ejemplo, el rango tiene 4 elementos, por lo que el bucle `for` se repite 4 veces:

```
I am in a loop!
I am in a loop!
I am in a loop!
I am in a loop!
```

Es importante tener en cuenta que la variable de bucle solo existe dentro del bloque de código del bucle. Intentar acceder a la variable de bucle fuera del bucle `for` provocará un error.

Aquí hay un ejemplo de cómo usar la variable de bucle dentro del cuerpo del bucle:

```kotlin
for (i in 1..4) {
  println("i = $i")
}
```

Si bien el ejemplo anterior usa números literales `1..4` para definir el rango, también podemos usar variables para definir los límites del rango de nuestro bucle, lo que nos da un control más dinámico sobre la iteración de nuestro bucle:

```kotlin
var num = 4
for (i in 1..num) {
  println("i = $i")
}
```

Ambos fragmentos de código producen la misma salida final:

```
i = 1
i = 2
i = 3
i = 4
```
-----

## Controlling the Iteration

A veces queremos contar hacia atrás, o contar de 2 en 2, ¡o ambas cosas! Usar ciertas funciones junto con el operador de rango normal (`..`) o en su lugar, puede mejorar las capacidades iterativas de nuestros bucles `for`. Las funciones `downTo`, `until` y `step` nos dan más control sobre un rango y, por lo tanto, más control sobre nuestros bucles.

La función `downTo` simplemente crea un grupo de valores en orden inverso, donde el límite inicial es mayor que el límite final. Para lograrlo, reemplaza el operador de rango (`..`) con `downTo`:

```kotlin
for (i in 4 downTo 1) {
  println("i = $i")
}
```

Podemos ver en la salida que el primer número en `i` es 4 y el último es 1:

```
i = 4
i = 3
i = 2
i = 1
```

La función `until` crea un rango ascendente, al igual que el operador `..`, pero excluye el límite superior:

```kotlin
for (i in 1 until 4) {
  println("i = $i")
}
```

El límite superior, 4, no está incluido en la salida:

```
i = 1
i = 2
i = 3
```

Hasta ahora, cada una de estas funciones, incluido el operador de rango (`..`), han contado hacia arriba o hacia abajo de uno en uno. La función `step` especifica la cantidad con la que cuentan estas funciones:

```kotlin
for (i in 1..8 step 2) {
  println("i = $i")
}
```

La variable de bucle `i` ahora aumenta en 2 por cada iteración. El último número de la salida es 7, ya que 2 pasos más desde allí sería 9, que está fuera del rango definido, `1..8`:

```
i = 1
i = 3
i = 5
i = 7
```

----

## Iterating Through Collections

En lugar de usar un rango, podemos usar la estructura de una colección como iterador. Un bucle `for` iterará a través de cada uno de los elementos de la colección, y la variable de bucle contendrá el valor del elemento actual. Nos centraremos en listas y conjuntos en este ejercicio, y en mapas en el siguiente.

```kotlin
val fruitList = listOf("apples", "oranges", "bananas")
 
for (fruit in fruitList) {
  println("I have $fruit.")
}
```

En este ejemplo, declaramos `fruit` como la variable de bucle y usamos `fruitList` como el iterador. Cada iteración del bucle asignará a `fruit` el valor del elemento actual de `fruitList`. Por lo tanto, el número de elementos contenidos en `fruitList` determina cuántas veces se ejecuta `println("I have $fruit.")`:

```
I have apples.
I have oranges.
I have bananas.
```

Cuando aprendimos sobre listas y conjuntos en la lección de colecciones, discutimos sus similitudes y diferencias. Una similitud adicional es que la sintaxis para iterar a través de una lista y un conjunto es la misma.

Cuando se itera a través de una colección, a menudo es útil saber en qué número de elemento o iteración nos encontramos. Para iterar a través de los índices de una colección, puedes usar su propiedad `indices`:

```kotlin
val fruitSet = setOf("apples", "oranges", "bananas")
 
for (setIndex in fruitSet.indices) {
  println("Index = $setIndex")
}
```

Aquí vemos la salida de los índices de `fruitSet`. Recuerda que el primer índice de una lista o un conjunto siempre es 0:

```
Index = 0
Index = 1
Index = 2
```

También podemos obtener tanto el índice como el elemento del iterador usando la función `withIndex()` de la colección. En este caso, necesitamos desestructurar la declaración de la variable de bucle declarando dos variables de bucle y encerrándolas entre paréntesis:

```kotlin
val fruitList = listOf("apples", "oranges", "bananas")
 
for ((listIndex, fruit) in fruitList.withIndex()) {
  println("$listIndex is the index of the element $fruit")
}
```

Usando `withIndex()` y la desestructuración, podemos acceder tanto al índice como al elemento de `fruitList`:

```
0 is the index of the element apples
1 is the index of the element oranges
2 is the index of the element bananas
```

----

## Iterating Through Maps

A diferencia de una lista o un conjunto, un mapa es una colección de entradas. Cuando se usa un mapa como iterador en un bucle `for`, podemos iterar a través de cada entrada del mapa, o solo a través de las claves, o solo a través de los valores.

Cuando se itera a través de un mapa, la variable de bucle contendrá una entrada por iteración. Se puede acceder a la clave y al valor de la entrada usando las propiedades `key` y `value`:

```kotlin
val myClothes = mapOf("Shirts" to 7, "Pairs of Pants" to 4, "Jackets" to 2)
 
for (itemEntry in myClothes) {
  println("I have ${itemEntry.value} ${itemEntry.key}")
}
```

Observa que para acceder a los atributos dentro de una plantilla de cadena (String template), rodeamos la variable de bucle y el atributo con llaves.

También podemos acceder a la clave y al valor de cada entrada mediante desestructuración, usando dos variables de bucle entre paréntesis. La primera variable es la clave de la entrada y la segunda es el valor:

```kotlin
val myClothes = mapOf("Shirts" to 7, "Pairs of Pants" to 4, "Jackets" to 2)
 
for ((itemName, itemCount) in myClothes) {
  println("I have $itemCount $itemName")
}
```

Ambos ejemplos tienen la misma salida:

```
I have 7 Shirts
I have 4 Pairs of Pants
I have 2 Jackets
```

Por último, podemos iterar solo a través de las claves o los valores de un mapa. Un mapa tiene las propiedades `keys` y `values`, cada una de las cuales puede usarse como iterador:

```kotlin
val myClothes = mapOf("Shirts" to 7, "Pairs of Pants" to 4, "Jackets" to 2)
 
println("KEYS")
for (itemName in myClothes.keys) {
  println(itemName)
}
 
println("\nVALUES")
for (itemCount in myClothes.values) {
  println(itemCount)
}
```

Aquí está la salida:

```
KEYS
Shirts
Pairs of Pants
Jackets

VALUES
7
4
2
```

----

## while Loop

Cuando repetimos código, es posible que no tengamos un rango o una colección definida que dicte el número de bucles que necesitamos ejecutar. En este caso, podemos recurrir a un bucle `while`, que repite el código mientras se cumpla una condición especificada. Si revisamos la animación del primer ejercicio, podemos ver que Codey usa un bucle `while` para caminar hasta que el tiempo ya no sea menor que 17.

Un bucle `while` es similar a una expresión `if` porque evalúa una condición y, cuando la condición es verdadera, se ejecuta un bloque de código. La principal diferencia es que el bucle `while` ejecutará el bloque de código, volverá a verificar la condición y repetirá el código si la condición sigue siendo verdadera. Si la condición es falsa, el bucle `while` finaliza y el programa continúa:

```kotlin
var myAge = 16

while (myAge < 20) {
  println("I am a teenager.")
  myAge += 1
}
println ("I am not a teenager.")
```

En este ejemplo, la condición del bucle `while` (`myAge < 20`) se evalúa inicialmente como `true`. Se ejecuta el cuerpo del bucle, que muestra algo de texto y se incrementa `myAge`. El bucle se repite hasta que `myAge` es igual a 20, momento en el cual el bucle termina y continúa ejecutando el código fuera del bucle:

```
I am a teenager. // myAge es 16
I am a teenager. // myAge es 17
I am a teenager. // myAge es 18
I am a teenager. // myAge es 19
I am not a teenager. // myAge es 20
```

Si `myAge` no se incrementara dentro del cuerpo del bucle, la condición del bucle siempre sería verdadera y se repetiría para siempre. Esto se conoce como **bucle infinito**. Los bucles infinitos provocan fallos en la aplicación u otros comportamientos no deseados, por lo que es importante evitarlos.

Cuando la condición de un bucle `while` nunca se evalúa como `true`, todo el bloque de código se omite:

```kotlin
var myAge = 44

while (myAge > 12 && myAge < 20) {
  println("I am a teenager.")
  myAge += 1
}
println ("I am not a teenager.")
```

Dado que hemos establecido el valor de `myAge` en 44 y la condición solo será verdadera cuando el valor esté entre 12 y 20, el bloque de código se omite y se ejecuta el `println()` final:

```
I am not a teenager.
```

----

## do..while Loop

Un bucle `do..while` es igual que un bucle `while`, excepto que la condición de iteración se verifica al final del cuerpo del bucle. Esto se conoce como un **bucle de condición de salida** y significa que el código en el cuerpo se ejecutará al menos una vez:

```kotlin
val myCondition = false

do {
  print("I loop once!")
} while(myCondition)
```

Este ejemplo muestra que, aunque tengamos una condición falsa, nuestro bucle ejecutará el cuerpo del bucle una vez:

```
I loop once!
```

Una razón para usar un bucle `do..while` es cuando la inicialización de la condición del bucle y los comandos de actualización son los mismos. Usando pseudocódigo, podemos comparar un bucle `while` con un bucle `do..while` usando cómo alguien podría decidir salir al exterior:

**Bucle `while`:**
```
¿Está soleado afuera?
mientras ( no esté soleado ) {
  ¿Está soleado afuera?
  Quédate dentro.
}
¡Sal afuera!
```

El ejemplo con `while` tiene una línea antes del bucle para verificar si está soleado afuera. Se entra al bucle si no está soleado y luego se verifica repetidamente si está soleado afuera. Cuando está soleado, el bucle termina y es hora de salir.

**Bucle `do..while`:**
```
haz {
  ¿Está soleado afuera?
  Quédate dentro.
} mientras ( no esté soleado )
¡Sal afuera!
```

El bucle `do..while` entra al bucle y comienza verificando si `isSunny` es verdadero. El cuerpo del bucle se repite si `isSunny` es falso.

La verificación adicional antes del bucle es innecesaria, ya que se realiza dentro del bucle de todos modos. Si tenemos que mirar afuera para obtener nuestra condición inicial del bucle y para actualizar la condición del bucle, bien podríamos tener eso como el cuerpo del bucle y verificar la condición al final.

Esta diferencia puede parecer sutil, pero si este bucle se ejecuta cientos o miles de veces por segundo, eliminar esa línea de código usando un bucle `do..while` podría ahorrar tiempo.

----

## Nested Loops

Bucles dentro de bucles, ¡oh cielos! Un **bucle anidado** representa un bucle colocado dentro del cuerpo de otro bucle, lo que da como resultado una mayor dimensión de iteraciones. Esta estrategia se usa más a menudo cuando queremos comparar el contenido de dos colecciones/iteradores diferentes.

A continuación se muestra un ejemplo de bucles anidados. El bucle externo itera a través del rango `1..2` y el bucle interno itera a través del rango `'A'..'C'`:

```kotlin
for (i in 1..2) {
  for (j in 'A'..'C') {
    println("Outer loop: $i - Inner loop: $j")
  }
}
```

Es importante entender que por cada iteración del bucle externo, el bucle interno ejecutará todas sus propias iteraciones.

En el ejemplo anterior, el bucle interno itera 3 veces por cada una de las 2 iteraciones del bucle externo. Por lo tanto, la instrucción `println()` se ejecuta 6 veces:

```
Outer loop: 1 - Inner loop: A
Outer loop: 1 - Inner loop: B
Outer loop: 1 - Inner loop: C
Outer loop: 2 - Inner loop: A
Outer loop: 2 - Inner loop: B
Outer loop: 2 - Inner loop: C
```

Cuando se comienza con bucles anidados, pueden parecer difíciles de manejar, pero hacen que las aplicaciones avanzadas sean fáciles de implementar.

-----

## Jump Expressions

Las expresiones de salto, `break` y `continue`, se utilizan para cambiar el comportamiento estándar de un bucle, ya sea saliendo de él anticipadamente o saltando una sola repetición.

La expresión `break` se usa para salir del bucle en una iteración particular:

```kotlin
val myNumbers = listOf(4, 8, 2, 9, 12, 7, 16, 10, 3)
for (num in myNumbers) {
  if (num > 9) {
    break
  }
  println(num)
}
```

En este ejemplo:
- El bucle `for` itera a través de la lista de `Int`.
- En cada iteración, el número actual `num` se evalúa para ver si es mayor que 9.
- Si es `false`, se imprime el número.
- Si es `true`, la expresión `break` sale del bucle.

Podemos ver en la siguiente salida que se detiene antes de imprimir el número 12:

```
4
8
2
9
```

La expresión `continue` saltará la ejecución actual del cuerpo del bucle, pero el bucle continuará si quedan iteraciones. Si la expresión `continue` se ejecuta, cualquier código restante en la iteración actual se omitirá:

```kotlin
val myNumbers = listOf(4, 8, 2, 9, 12, 7, 16, 10, 3)
for (num in myNumbers) {
  if (num > 9) {
    continue
  }
  println(num)
}
```

Este ejemplo es el mismo que el anterior, pero reemplazamos `break` con `continue`. La siguiente salida omite todos los valores de `num` mayores que 9, mientras imprime el resto de los números:

```
4
8
2
9
7
3
```

El primer ejemplo imprime los elementos en `myNumbers` hasta que se encuentra uno mayor que 9. El segundo ejemplo imprime todos los elementos en `myNumbers`, excepto aquellos que son mayores que 9.

Dado que las expresiones de salto generalmente están contenidas dentro de expresiones `if`, nos brindan la capacidad de agregar una condición para salir de un bucle `for` o usar otra condición para salir de un bucle `while`.

No te preocupes si todo esto no se asimila de inmediato; sigue practicando con cada expresión de salto y rápidamente comprenderás cómo y cuándo se usa cada una.

----

## Labeled Jump Expressions

"¿Cómo usaríamos las expresiones de salto dentro de bucles anidados?" Buena pregunta. La respuesta son las **expresiones de salto etiquetadas**. Al agregar una etiqueta al bucle externo de bucles anidados, podemos ejecutar una expresión de salto desde el bucle interno y hacer que actúe sobre el bucle externo etiquetado.

Primero, necesitamos darle un nombre a nuestra etiqueta. En el siguiente ejemplo, elegimos el nombre `rps`. El bucle externo se marca usando nuestra etiqueta `rps` seguida del símbolo `@`:

```kotlin
val game = listOf("Rock", "Paper", "Scissor")

rps@ for (p1 in game) {
  for (p2 in game) {
    if (p1 == "Paper") {
      break@rps
    }
    println("$p1 vs. $p2")
  }
}
```

El bucle interno tiene una expresión `if` que contiene una expresión `break` con `@rps` agregado. Cuando la condición `p1 == "Paper"` es verdadera, se ejecuta `break@rps` y sale del bucle externo:

```
Rock vs. Rock
Rock vs. Paper
Rock vs. Scissor
```

Ahora veamos qué sucede en nuestro ejemplo cuando cambiamos la expresión `break` por un `continue`:

```kotlin
val game = listOf("Rock", "Paper", "Scissor")

rps@ for (p1 in game) {
  for (p2 in game) {
    if (p1 == "Paper") {
      continue@rps
    }
    println("$p1 vs. $p2")
  }
}
```

Lo que vemos ahora es que el bucle externo omite una iteración cuando `p1 == "Paper"`, pero se reanuda con la iteración final cuando `p1` es igual a `"Scissor"`:

```
Rock vs. Rock
Rock vs. Paper
Rock vs. Scissor
Scissor vs. Rock
Scissor vs. Paper
Scissor vs. Scissor
```

Las expresiones de salto etiquetadas nos dan más control sobre los bucles anidados para ayudarnos a lograr el comportamiento deseado de nuestro código. Usarlas puede ser complicado, pero una vez que se comprenden, pueden ser muy poderosas.

-----

## Resumen en 5 líneas

1. `for (i in 1..5)` recorre un rango; `until`, `downTo` y `step` controlan cómo avanza.
2. `for (x in lista)` recorre una colección y `for ((clave, valor) in mapa)` recorre un map.
3. `while` repite mientras la condición sea `true`; `do..while` ejecuta el bloque al menos una vez antes de evaluar.
4. `break` sale del bucle y `continue` salta a la siguiente iteración.
5. Con etiquetas (`externo@`) puedes usar `break@externo` o `continue@externo` en bucles anidados.

-----

## Siguiente lección

Terminaste esta carpeta. Sigue con la siguiente: [Funciones](../05-funciones/README.md).
