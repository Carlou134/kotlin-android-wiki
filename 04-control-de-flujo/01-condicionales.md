# Conditional Expressions

## En una frase

Los condicionales deciden qué parte del código se ejecuta según si una condición es `true` o `false`; en Kotlin, `if` y `when` además pueden devolver un valor.

-----

## Antes de empezar

Conviene que ya sepas:

* El tipo `Boolean`: [Tipos y variables](../02-tipos-y-variables/01-tipos-y-variables.md).
* Los valores que vas a comparar: [Números](../02-tipos-y-variables/03-numeros.md).

Palabras nuevas (también están en el [Glosario](Glosario.md)):

* **Condicional:** Expresión que evalúa `true` o `false` para decidir qué código se ejecuta.
* **Condicional anidado:** Un condicional dentro de otro.
* **Operador de comparación:** Compara dos valores y devuelve un `Boolean`: `>`, `<`, `>=`, `<=`, `==` y `!=`.
* **Operador lógico:** Combina o niega condiciones: `&&` (y), `||` (o) y `!` (no).
* **Rango (`..`):** Sucesión consecutiva de valores, como `1..10`. Con `in` preguntas si un valor está dentro.
* **`when`:** Expresión que compara un valor contra varios casos. Reemplaza cadenas largas de `if/else`.

-----

## Introduction to Conditionals

Cuando salimos de aventuras, a veces el camino es sencillo y otras veces, el camino se divide y debemos elegir qué rumbo tomar. Los **condicionales** son como bifurcaciones en el camino: determinan la dirección que toma nuestro programa.

Un condicional es una expresión que evalúa valores verdaderos o falsos (`true` o `false`) para controlar el flujo del código en un programa.

Cuando hablamos de Kotlin en términos del orden de ejecución, dijimos que el código Kotlin se compila y ejecuta en orden descendente (de arriba abajo). Esto sigue siendo cierto; sin embargo, una instrucción condicional se puede usar para determinar qué instrucciones se ejecutan y qué instrucciones se ignoran.

En esta lección, aprenderemos a controlar el flujo de ejecución del código en nuestros programas Kotlin.

----
## Condicionales en Kotlin, explicado simple

Un **condicional** permite que el programa tome decisiones.

En vez de ejecutar siempre el mismo código, Kotlin puede evaluar una condición y decidir qué bloque ejecutar.

La estructura básica es `if`:

```kotlin
if (condicion) {
  // Se ejecuta si la condición es true
}
```

La condición siempre debe producir un valor booleano: `true` o `false`.

### `if` y `else`

Usamos `else` cuando queremos definir qué pasa si la condición no se cumple.

```kotlin
val email = "persona@email.com"

if (email.contains("@")) {
  println("El email tiene formato válido")
} else {
  println("El email no es válido")
}
```

En este caso, `contains("@")` devuelve `true` si el texto contiene `@`. Si no lo contiene, devuelve `false`.

### Múltiples condiciones con `else if`

Cuando hay más de dos posibilidades, usamos `else if`.

```kotlin
val password = "secreto123"

if (password.length >= 8) {
  println("Contraseña fuerte")
} else if (password.length >= 6) {
  println("Contraseña media")
} else {
  println("Contraseña no segura")
}
```

El orden importa. Kotlin evalúa desde arriba hacia abajo y se queda con la primera condición verdadera.

### `if` como expresión

En Kotlin, `if` también puede devolver un valor. Eso permite guardar el resultado en una variable.

```kotlin
val nivelSeguridad = if (password.length >= 8) {
  "nivel alto"
} else if (password.length >= 6) {
  "nivel medio"
} else {
  "nivel bajo"
}

println("El nivel de seguridad es $nivelSeguridad")
```

Acá `if` no solo ejecuta código: también produce un resultado.

Importante: si usás `if` como expresión, todas las ramas deben devolver un valor compatible. No mezcles, por ejemplo, un `String` en una rama y un `Int` en otra si esperás guardar el resultado como texto.

### `if` dentro de funciones

También podemos usar `if` para decidir qué valor retorna una función.

```kotlin
fun clasificarEmail(email: String): String {
  return if (email.contains("admin")) {
    "administrador"
  } else if (email.contains("support")) {
    "soporte"
  } else {
    "usuario normal"
  }
}
```

Esta función recibe un email y devuelve una clasificación según su contenido.

### Buenas prácticas

Escribir condicionales no es solo hacer que el código funcione. Es hacer que la decisión sea fácil de entender.

- Usá nombres claros para las variables booleanas.
- Evitá anidar demasiados `if` dentro de otros `if`.
- Si hay muchas condiciones relacionadas, considerá usar `when`.
- Mantené cada condición lo más simple posible.

**Idea clave:** los condicionales le dan inteligencia al programa. Permiten que el código reaccione según los datos que recibe.

----

Los operadores lógicos y de comparación son la base para tomar decisiones en Kotlin. Siempre producen un valor `Boolean`: `true` o `false`.

Por ejemplo, puedes guardar el resultado de una condición en una variable:

```kotlin
val emailsRecibidos = 15
val tieneEmails = emailsRecibidos > 0

println("Tiene emails: $tieneEmails") // Imprime: Tiene emails: true
```

Ese resultado booleano luego puede usarse dentro de un `if` para decidir qué bloque de código se ejecuta.

----

## if Expression

Una expresión `if` es un condicional que ejecuta un bloque de código solo cuando un valor booleano especificado devuelve `true` (verdadero).

Supongamos que tenemos una variable booleana llamada `raining` (lloviendo):

```kotlin
var raining = true
```

Podemos usar una expresión `if` para evaluar el valor de `raining` y determinar si ejecutamos o no algún código:

```kotlin
if (raining) {
  println("Bring an umbrella.")
}
```

La expresión comienza con la palabra clave `if`.
Un par de paréntesis contiene una condición (`raining`) que se evalúa como `true` o `false` (verdadero o falso).
Después de la condición viene un par de llaves `{ }` que contiene un bloque de código.
El código contenido dentro del bloque de código de la expresión `if` solo se ejecutará si la condición es `true`; de lo contrario, ese bloque de código se ignorará.

Cuando ejecutamos el programa anterior, obtenemos la siguiente salida:

```
Bring an umbrella.
```

El valor de `raining` dentro de la expresión `if` es `true`; por lo tanto, la instrucción `println("Bring an umbrella.")` se ejecuta.

----

## if/else Expression

A menudo, acompañando a la expresión `if` viene otro condicional llamado expresión `else`. La expresión `else` ejecuta un bloque de instrucciones cuando la condición en la expresión `if` tiene un valor `false` (falso).

Por ejemplo:

```kotlin
var isHerbivore = false
var diet: String

if (isHerbivore) {
  diet = "vegetables"
} else {
  diet = "meat"
}
println("This animal can eat $diet.")

// Imprime: This animal can eat meat.
```

La expresión `else` no va seguida de una condición, ya que solo ejecutará código si la condición de la expresión `if` es `false`.

Esta expresión siempre se coloca como el último condicional y no puede existir sin una expresión `if` que la acompañe.

----

## Comparison Operators

Para crear condiciones más complejas que nuestros condicionales puedan evaluar, podemos usar **operadores de comparación**. Los operadores de comparación evalúan la relación entre dos valores para devolver una expresión como `true` o `false`.

Hay varios operadores de comparación que podemos usar en Kotlin: `<`, `>`, `>=`, y `<=`.

El **operador menor que** (`<`) evalúa como `true` si el valor del lado izquierdo de `<` es más pequeño que el valor del lado derecho:

```kotlin
var username = "beetle"

if (username.length < 10) {
  println("Please choose a longer username.")
}
// Imprime: Please choose a longer username.
```

Con el **operador mayor que** (`>`), si el valor del lado izquierdo del símbolo `>` es más grande que el valor del lado derecho, la expresión devuelve `true`:

```kotlin
var username = "beetlejuice_beetlejuice_beetlejuice"

if (username.length > 25) {
  println("Please choose a shorter username.")
}
// Imprime: Please choose a shorter username.
```

El **operador menor o igual que** (`<=`) devuelve `true` solo si el valor de la izquierda es menor o igual que el valor de la derecha:

```kotlin
var liquidTemp = 27 // Fahrenheit

if (liquidTemp <= 32) {
  println("The liquid will freeze.")
} else {
  println("The liquid will remain liquid.")
}
// Imprime: The liquid will freeze.
```

El **operador mayor o igual que** (`>=`) devolverá `true` si el valor de la izquierda es mayor o igual que el valor de la derecha:

```kotlin
var liquidTemp = 50 // Fahrenheit

if (liquidTemp >= 212) {
  println("The liquid will vaporize.")
} else {
  println("The liquid will remain liquid.")
}
// Imprime: The liquid will remain liquid.
```

Ejemplo con una bandeja de entrada:

```kotlin
val emailsRecibidos = 15
val emailsEnviados = 10
val limiteEmails = 20

val tieneEmails = emailsRecibidos > 0
val excedeEnviados = emailsEnviados >= limiteEmails

println("Tiene emails: $tieneEmails")       // true
println("Excede enviados: $excedeEnviados") // false
```

La idea importante es esta: una comparación no devuelve un número, devuelve un `Boolean`.

----

## Equality and Inequality Operators

Otra forma de comparar dos valores es con los **operadores de igualdad**.

Podemos usar el **operador de igualdad** (`==`) para verificar si el valor del lado derecho del operador es igual al valor del lado izquierdo. Si los valores son equivalentes, la expresión devolverá `true`; de lo contrario, devolverá `false`:

```kotlin
var userPassword = "secret884"
var attemptedPassword = "secret884"

if (userPassword == attemptedPassword) {
  println("The password you entered is correct.")
}
```

Este fragmento de código mostrará `"The password you entered is correct."` porque `userPassword` y `attemptedPassword` tienen el mismo valor.

Por otro lado, el **operador de desigualdad** (`!=`) se usa para determinar si los valores a cada lado del operador `!=` son diferentes. Esta expresión devuelve `true` cuando los dos valores que se comparan no son iguales entre sí.

Podemos usar `!=` para verificar si una reunión de 9 personas puede repartir uniformemente 16 porciones de pizza entre ellas:

```kotlin
var slicesOfPizza = 16
var numPeople = 9

if (slicesOfPizza % numPeople != 0) {
  println("The pizza cannot be split evenly.")
}
// Imprime: The pizza cannot be split evenly.
```

Usando el módulo `%`, podemos averiguar si dividir 16 entre 9 tiene un resto distinto de 0. Como el valor del resto no es igual a 0, se muestra `"The pizza cannot be split evenly."` en la terminal.

Cuidado con esta diferencia, porque es FUNDAMENTAL:

```kotlin
val cantidad = 10      // = asigna un valor
val esDiez = cantidad == 10 // == compara valores
```

- `=` sirve para asignar.
- `==` sirve para comparar.
- `!=` sirve para verificar que dos valores sean diferentes.

Otro ejemplo:

```kotlin
val emailsRecibidos = 15
val emailsEnviados = 10

val sonIguales = emailsRecibidos == emailsEnviados
val sonDiferentes = emailsRecibidos != emailsEnviados

println(sonIguales)     // false
println(sonDiferentes)  // true
```

----

## else-if Expressions

Una expresión `else-if` se puede usar para agregar una gran cantidad de condicionales más para que nuestro programa evalúe.

Digamos que creamos un programa en Kotlin que determina qué usaremos según la temperatura exterior. Si solo usamos una expresión `if/else`, nuestras opciones son limitadas:

```kotlin
var temp = 60

if (temp > 65) {
  println("Wear a t-shirt")
} else {
  println("Wear a winter coat")
}
// Imprime: Wear a winter coat
```

En la vida real, usar un abrigo de invierno con una temperatura de 60 grados puede hacer que una persona se sobrecaliente. Necesitamos usar algo entre un abrigo pesado y una camiseta, como una chaqueta ligera. Podemos colocar una expresión `else-if` en nuestro código para representar esta situación intermedia:

```kotlin
var temp = 60

if (temp > 65) {
  println("Wear a t-shirt")
} else if (temp > 45) {
  println("Wear a light coat") 
} else {
  println("Wear a winter coat")
}
// Imprime: Wear a light coat
```

Examinemos esta instrucción `else-if`:

- La expresión `else-if` debe venir después de una expresión `if` pero antes de la expresión `else`.
- La expresión `else-if` contiene su propia condición colocada entre paréntesis: `temp > 45`.
- La instrucción `println("Wear a light coat")` se ejecuta solo si la condición en la expresión `if` es `false` y la condición dentro de la expresión `else-if` es `true`. Si ambas condiciones son `false`, se ejecutarán las instrucciones del siguiente condicional, la expresión `else`.

Una expresión `if-else` puede contener tantas expresiones `else-if` como sea necesario. Cuando se usan múltiples instrucciones `else-if`, el orden en que aparecen es importante. En el momento en que el programa evalúa uno de los condicionales como `true`, el programa ejecutará ese bloque de código y no evaluará ninguno de los condicionales que aparezcan después.

-----

## Logical Operators

Los **operadores lógicos** evalúan la relación entre dos o más expresiones booleanas y devuelven un valor `true` o `false`. Hay varios operadores que podemos usar: `&&`, `||` y `!`.

El **operador AND lógico** (`&&`) devuelve `true` si y solo si las expresiones en ambos lados del operador devuelven `true`.

```kotlin
var isSunny = true
var temp = 85

if (isSunny && temp > 80) {
  println("Wear sunglasses.")
}
// Imprime: Wear sunglasses.
```

Dado que `isSunny` es `true` y `temp` tiene un valor superior a 80, la expresión dentro de la condición se vuelve `true`, por lo que se muestra `Wear sunglasses.` en la terminal.

El **operador OR lógico** (`||`) solo devolverá `true` si al menos una de las expresiones booleanas que se comparan tiene un valor `true`.

```kotlin
var isSnowing = false
var temp = 38

if (isSnowing || temp < 40 ) {
  println("Wear a scarf.")
}
// Imprime: Wear a scarf.
```

Aunque el primer valor `isSnowing` es `false`, la expresión completa devolverá `true` porque la otra expresión booleana, `temp < 40`, es `true`.

El **operador NOT lógico**, representado por `!`, evaluará el valor de verdad de una expresión booleana y luego devolverá su inverso. Por ejemplo:

```kotlin
var isSnowing = true
println(!isSnowing) // Imprime: false
```

Incluso podemos agregar `!` antes de una expresión de múltiples partes. Al agregar el operador `!` a nuestro ejemplo de código anterior, la expresión booleana ahora devuelve un valor `false`:

```kotlin
var isSnowing = false
var temp = 38

if (!(isSnowing || temp < 40)) {
  println("Wear a scarf.")
}
// Este programa no tendrá ninguna salida.
```

**Nota:** Las tablas de verdad que se muestran en el lado derecho de la pantalla muestran los diversos resultados de las expresiones booleanas evaluadas mediante operadores lógicos.

-----

### Validar un email con `&&`

Los `String` en Kotlin tienen funciones que también devuelven booleanos. Por ejemplo, `contains()` verifica si un texto contiene otro texto.

```kotlin
val email = "correo@example.com"

val tieneArroba = email.contains("@")
val tienePunto = email.contains(".")
val esEmailValido = tieneArroba && tienePunto

println(esEmailValido) // true
```

`&&` solo devuelve `true` cuando todas las condiciones son verdaderas. Si una falla, el resultado completo es `false`.

### Detectar una contraseña problemática con `||`

```kotlin
val password = "1234"

val passwordCorto = password.length < 6
val passwordLargo = password.length > 20
val passwordProblematico = passwordCorto || passwordLargo

println(passwordProblematico) // true
```

`||` devuelve `true` cuando al menos una condición se cumple. En este caso, la contraseña es problemática porque es demasiado corta.

### Negar una condición con `!`

```kotlin
val esEmailInvalido = !esEmailValido

println(esEmailInvalido) // false
```

`!` invierte el valor booleano: `true` se vuelve `false`, y `false` se vuelve `true`.

-----

## Ejemplo práctico de validación

Veamos un caso más real: aceptar un correo solo si cumple varias reglas.

Reglas:

- El email debe contener `@` y `.`.
- El email no debe contener la palabra `test`.
- El asunto no debe contener `gratis` ni `urgente`.

```kotlin
val email = "hola@dominio.com"
val asunto = "Reunión del lunes"

val esEmailValido = email.contains("@") &&
                    email.contains(".") &&
                    !email.contains("test")

val noEsSpam = !asunto.contains("gratis") &&
               !asunto.contains("urgente")

val esAceptable = esEmailValido && noEsSpam

println(esAceptable) // true
```

Aquí hay dos ideas importantes:

- `&&` encadena condiciones que deben cumplirse juntas.
- `!` excluye casos que no queremos aceptar.

El resultado final, `esAceptable`, queda claro porque cada variable representa una parte de la regla. ESO es escribir condiciones bien pensadas, no solo juntar operadores porque sí.

-----

## Order of Evaluation

Podemos combinar expresiones que contengan operadores lógicos para crear expresiones booleanas aún más complejas. Al combinar operadores lógicos, el orden de evaluación es el siguiente:

1. Expresiones booleanas contenidas entre paréntesis
2. NOT - `!`
3. AND - `&&`
4. OR - `||`

Observa el siguiente fragmento de código:

```kotlin
var isSnowing = true
var isSunny = false
var temp = 38
 
if (!isSunny && (temp > 40 || isSnowing)) {
  println("Wear a coat.")
} else {
  println("Don't wear a coat.")
}
```

Analicemos el orden en que se evalúa esta expresión. Aquí está la expresión booleana que estamos evaluando:

```
!isSunny && (temp > 40 || isSnowing)
```

`(temp > 40 || isSnowing)` se evalúa primero porque la expresión está contenida entre paréntesis. Esta expresión da como resultado `true`, haciendo que nuestra expresión se vea así:

```
!isSunny && true
```

`!isSunny` se evalúa a continuación debido al operador NOT. Esta expresión es `true` porque niega el valor de `isSunny`:

```
true && true
```

Finalmente, evaluamos la expresión usando el operador lógico AND (`&&`). Dado que ambos lados del operador tienen un valor `true`, la expresión dentro de nuestro `if` es `true` y obtenemos la salida final:

```
Wear a coat.
```

----

## Nested Conditionals

Similar al concepto de un "sueño dentro de un sueño" de la película *Inception*, un condicional puede existir dentro de otro condicional. A esto se le llama **condicional anidado**.

Al anidar un condicional dentro de otro condicional, podemos lograr soluciones lógicas más complejas y precisas.

Por ejemplo, ¿recuerdas Conditional Creek? Podemos usar un condicional anidado para replicar esta situación:

```kotlin
var sailColor = "red"
var sailPattern = "striped"

if (sailColor == "red") {
  println("Follow the lower river.")
  if (sailPattern == "striped") {
    println("Then take the right path.")
  } else {
    println("Then take the left path.")
  }
} else {
  println("Take the higher river.")
}
```

El primer condicional verifica el valor verdadero de `sailColor`, mientras que el condicional anidado verifica el valor verdadero de `sailPattern`. Este programa mostrará lo siguiente:

```
Follow the lower river.
Then take the right path.
```

----

## when Expressions

Otra expresión que controla el flujo de ejecución en un programa es la expresión `when`.

Una expresión `when` está compuesta por múltiples ramas (branches) que contienen valores o argumentos para verificar. Si el valor de la variable que se está evaluando coincide con uno de los argumentos de las ramas, se ejecutarán las instrucciones contenidas en esa rama.

Por ejemplo:

```kotlin
var lightColor = "red"

when (lightColor) {
  "green" -> println("Go.")
  "yellow" -> println("Slow down.")
  "red" -> println("Stop.")
  else -> println("Not a valid traffic light color.")
}
// Imprime: Stop.
```

- La palabra clave `when` va seguida de una variable entre paréntesis.
- La configuración de cada rama comienza con un valor a verificar, una flecha (`->`) y luego una instrucción a ejecutar.
- El `else` al final de una expresión `when` es una adición opcional que se puede usar para capturar cualquier valor inesperado o adicional que pueda contener la variable evaluada.

En el ejemplo anterior, el valor de `lightColor` era `"red"`, por lo que se ejecutó la instrucción `println("Stop.")`.

Los programadores suelen usar expresiones `when` en lugar de condicionales `if/else` o `else-if` debido a que su sintaxis es significativamente más corta. Por ejemplo, podemos crear una expresión `when` que evalúe múltiples expresiones:

```kotlin
var num = 19
when {
  num < 0 -> println("$num is negative.")
  num == 0 -> println("$num is zero.")
  num > 0 -> println("$num is positive.")
  else -> println("Not a valid number.")
}
// Imprime: 19 is positive.
```

Observa cómo en el ejemplo anterior no colocamos un valor entre paréntesis después de la palabra clave `when` y, en cambio, colocamos expresiones completas dentro de los argumentos de las ramas.

Para explorar otras formas de implementar una expresión `when` en nuestros programas, consulta la Guía de Lenguaje de Kotlin.

-----

## `when` en Kotlin, explicado simple

Cuando un programa tiene muchas condiciones, varios `if/else` seguidos pueden volverse difíciles de leer. Para esos casos, Kotlin ofrece `when`.

`when` sirve para evaluar varias posibilidades y ejecutar la primera rama que coincida.

### `when` sin argumento

Usamos `when` sin argumento cuando cada rama tiene su propia condición.

```kotlin
val email = "administrador@empresa.com"

when {
  email.contains("admin") -> println("administrador")
  email.contains("support") -> println("soporte")
  else -> println("usuario normal")
}
```

El orden importa: Kotlin revisa de arriba hacia abajo y ejecuta la primera condición verdadera.

### `when` como expresión

`when` también puede devolver un valor y guardarlo en una variable.

```kotlin
val tipoDeUsuario = when {
  email.contains("admin") -> "administrador"
  email.contains("support") -> "soporte"
  else -> "usuario normal"
}

println("El tipo de usuario es $tipoDeUsuario")
```

Acá cada rama devuelve un `String`. Lo ideal es que todas las ramas devuelvan el mismo tipo para mantener el código simple y predecible.

### `when` con argumento

También podemos pasarle un valor a `when`. En ese caso, Kotlin compara ese valor contra cada rama.

```kotlin
val proveedor = readLine()

when (proveedor) {
  "Gmail" -> println("Google Mail")
  "Yahoo" -> println("Yahoo Mail")
  "Outlook" -> println("Microsoft Outlook")
  else -> println("Otro proveedor")
}
```

Esta forma se parece al `switch` de otros lenguajes, pero `when` es más flexible.

### Combinar condiciones en `when`

Podés usar operadores lógicos dentro de las ramas.

```kotlin
val email = "persona@gmail.com"
val esEmailValido = email.contains("@") && email.contains(".com")
val esGmail = email.contains("gmail")

when {
  esEmailValido && esGmail -> println("Es un correo válido de Gmail")
  esEmailValido -> println("Es un correo válido")
  else -> println("Es un correo no válido")
}
```

La condición más específica va primero. Si pusieras `esEmailValido` arriba, Kotlin entraría ahí antes de revisar si además es Gmail.

### Ejemplo: evaluar una contraseña

```kotlin
fun evaluarPassword(password: String): String {
  return when {
    password.length >= 8 -> "fuerte"
    password.length >= 6 -> "media"
    else -> "débil"
  }
}

fun main() {
  val password = readLine() ?: ""
  println("Es una contraseña ${evaluarPassword(password)}")
}
```

**Idea clave:** usá `if/else` para decisiones simples y `when` cuando tengas varias condiciones. Código claro primero, SIEMPRE.

-----

## Range

En Kotlin, los rangos proporcionan una herramienta poderosa que representa una sucesión consecutiva de valores y que a menudo se puede usar junto con condicionales.

En Kotlin, los rangos se crean usando el operador `..`:

```
ValorInicial..ValorFinal
```

El rango comenzará en el valor que aparece antes de `..` y continuará hasta alcanzar el valor que aparece después de `..`.

Podemos verificar si los valores están contenidos dentro de un rango específico con la ayuda de la palabra clave `in`:

```kotlin
var num = 5

if (num in 1..10) {
  println("This value is between 1 and 10.")
}
```

La palabra clave `in` se usa para verificar si un valor existe dentro de una colección o rango. El programa anterior verifica si `num` existe dentro del rango del 1 al 10.

También podemos verificar si un carácter, o `Char`, aparece dentro de un rango:

```kotlin
var letter = 'c'

when (letter) {
  in 'a'..'m' -> println("Letter is in 1st half of alphabet.")
  in 'n'..'z' -> println("Letter is in 2nd half of alphabet.")
  else -> println("Not a valid value")
}
```

**Nota:** Los rangos de letras distinguen entre mayúsculas y minúsculas (son case-sensitive).

-----

## Review

Excelente trabajo por llegar al final de esta lección. Repasemos el contenido que aprendimos:

- Las **expresiones condicionales** controlan el flujo de ejecución en un programa.
- Una expresión `if` ejecuta un bloque de código cuando el valor de su condición es verdadero (`true`).
- Una expresión `else` ejecuta código cuando ninguna de las expresiones anteriores tiene un valor verdadero.
- Los **operadores de comparación** como `>`, `<`, `>=` y `<=` se usan para comparar dos valores y determinar un valor verdadero o falso.
- El operador `==` verifica si dos valores son equivalentes, mientras que el operador `!=` verifica si dos valores **no** son equivalentes.
- El **operador lógico AND** (`&&`) devuelve `true` cuando dos expresiones booleanas que se comparan son ambas verdaderas.
- El **operador lógico OR** (`||`) devuelve `true` cuando al menos una expresión booleana que se compara es verdadera.
- El **operador lógico NOT** (`!`) niega el valor de una expresión booleana.
- Funciones como `contains()` y propiedades como `length` ayudan a crear validaciones con `String`.
- Una condición bien escrita puede guardarse en una variable booleana para que el código sea más claro.
- Las expresiones `else-if` se usan para agregar más condiciones a una sentencia `if/else` tradicional.
- Un **condicional anidado** es un condicional que aparece dentro de otro condicional.
- Una expresión `when` evalúa el valor de una variable o expresión para dictar el flujo de un programa.
- El **operador de rango** `..` crea una sucesión consecutiva de valores.

-----

## Resumen en 5 líneas

1. `if` ejecuta un bloque cuando la condición es `true`; `else if` agrega más condiciones y `else` cubre el resto.
2. Comparas con `>`, `<`, `>=`, `<=`, `==` y `!=`; el resultado siempre es un `Boolean`.
3. Combinas condiciones con `&&` (ambas), `||` (al menos una) y `!` (negación).
4. `when` reemplaza cadenas largas de `if/else` evaluando un valor contra varios casos.
5. El operador `..` crea rangos (`1..10`) que puedes usar con `in`, dentro de un `if` o de un `when`.

-----

## Siguiente lección

Sigue con: [Bucles](02-bucles.md).
