# Functions

## En una frase

Una función es un bloque de código con nombre que recibe datos (argumentos), hace una tarea y puede devolver un resultado.

-----

## Antes de empezar

Conviene que ya sepas:

* La lógica que vas a encapsular: [Condicionales](../04-control-de-flujo/01-condicionales.md).
* Recorrer datos dentro de una función: [Bucles](../04-control-de-flujo/02-bucles.md).

Palabras nuevas (también están en el [Glosario](Glosario.md)):

* **Alcance (scope):** Zona del programa donde una variable existe. Una variable declarada dentro de una función no existe fuera de ella.
* **Argumento:** Dato que le pasas a una función al llamarla.
* **Argumento nombrado:** Argumento que indicas por su nombre al llamar: `saludar(name = "Ana")`.
* **Argumento predeterminado:** Valor que toma un parámetro si no le pasas ninguno: `name: String = "Customer"`.
* **Función:** Bloque de código reutilizable con nombre, que se ejecuta cuando lo llamas.
* **Función de expresión única:** Función de una sola expresión escrita con `=` y sin llaves: `fun cuadrado(n: Int) = n * n`.
* **Lambda:** Función sin nombre escrita entre llaves que se puede guardar en una variable o pasar como argumento: `{ a: Int -> a * 2 }`.
* **Tipo de retorno:** Tipo del valor que devuelve una función, escrito después de `:` en su encabezado.

-----

## Introduction to Functions

Ahora que entendemos conceptos importantes de programación como el flujo de control, las colecciones y los bucles, podemos aprender a usar funciones para hacer que nuestros programas sean más eficientes. Una **función** es un bloque de código reutilizable que podemos llamar a lo largo de nuestro programa para completar una determinada tarea.

Es posible que reconozcas el término "funciones", ya que ya hemos implementado algunas en este curso. La función `println()`, que hemos usado para mostrar valores, así como la función `main()`, en la que se ha escrito todo nuestro código hasta ahora, son funciones definidas por la biblioteca estándar de Kotlin. En esta lección, saldremos de la función `main()` y escribiremos nuestras propias funciones diseñadas por el usuario.

Las funciones diseñadas por el usuario son funciones que nosotros, los desarrolladores, creamos en nuestros programas para lograr una tarea específica. Hay muchas formas de crear y usar funciones, lo que las convierte en una herramienta extremadamente versátil.

En esta lección, centraremos nuestros esfuerzos en cómo escribir e invocar funciones diseñadas por el usuario.

--------
## Funciones en Kotlin, explicado simple

Una **función** es una parte del programa que agrupa instrucciones para realizar una tarea concreta.

En vez de escribir todo dentro de `main()`, dividimos el código en funciones pequeñas. Eso hace que el programa sea:

- más claro;
- más fácil de leer;
- más fácil de mantener;
- más fácil de corregir o modificar.

La idea es simple: **si una parte del código tiene una responsabilidad clara, puede convertirse en una función**.

```kotlin
fun mostrarBienvenida() {
  println("Bienvenido al gestor de emails")
}

fun main() {
  mostrarBienvenida()
}
```

La función se declara con `fun`, pero no se ejecuta sola. Para usarla, hay que **invocarla** escribiendo su nombre con paréntesis.

### Parámetros y valores por defecto

Una función puede recibir datos para personalizar lo que hace. Esos datos se llaman **parámetros**.

```kotlin
fun saludarUsuario(nombre: String = "Usuario") {
  println("Hola, $nombre, tienes nuevos emails")
}

fun main() {
  saludarUsuario("Juan")
  saludarUsuario()
}
```

En este caso, `nombre` tiene un valor por defecto. Si enviamos `"Juan"`, usa ese valor. Si no enviamos nada, usa `"Usuario"`.

### Retornar valores

Una función también puede devolver un resultado usando `return`.

```kotlin
fun leerEmail(): String {
  print("Ingresa tu email: ")
  val email = readLine()?.trim() ?: "sin correo"
  return email
}
```

El `: String` indica que la función devuelve un texto. Ese valor puede guardarse en una variable o pasarse a otra función.

```kotlin
fun main() {
  val email = leerEmail()
  println("Email registrado: $email")
}
```

### Parámetros nombrados

Cuando una función tiene varios parámetros, Kotlin permite escribir el nombre de cada uno al llamarla. Esto mejora la claridad y evita confusiones con el orden.

```kotlin
crearEmail(
  mensaje = "Hola, ¿cómo estás?",
  destinatario = "persona@email.com",
  asunto = "Saludo"
)
```

### Scope o alcance

Las variables creadas dentro de una función solo existen dentro de esa función.

```kotlin
fun calcularTotal() {
  val total = 100
  println(total)
}
```

La variable `total` no puede usarse fuera de `calcularTotal()`. Esto ayuda a mantener el código ordenado y evita que una función interfiera con otra.

### Encadenar funciones

Si una función devuelve un valor, ese resultado puede usarse directamente en otra función.

```kotlin
println(leerEmail())
```

Primero se ejecuta `leerEmail()`, devuelve un texto, y luego `println()` imprime ese resultado.

**Idea clave:** las funciones sirven para dividir un problema grande en partes pequeñas, claras y reutilizables. No son decoración: son una herramienta para ordenar el pensamiento y el código.

--------

## Creating and Calling a Function

Para declarar una función, necesitaremos entender la anatomía de una función básica.

Un **encabezado de función** contiene información valiosa sobre una función, incluidos su nombre, argumentos y su tipo de retorno.

```kotlin
fun nombreFuncion(argName: Tipo): TipoRetorno {
  // Cuerpo de la función
}
```

- La palabra clave `fun` se usa para declarar una función.
- El **nombre de la función** se usa para llamar a la función a lo largo del programa.
- Dentro de los paréntesis se encuentran los **argumentos**: piezas de datos que se le pasan a la función.
- El **tipo de retorno** declara el tipo de dato que devolverá la función; también es opcional incluirla.
- Después del encabezado de la función viene el **cuerpo de la función**, donde se encuentran las instrucciones.

Cubriremos los argumentos y los tipos de retorno más adelante en esta lección. Por ahora, centraremos nuestra atención en crear una función básica que muestre un valor cada vez que la llamemos.

Creemos una función llamada `greeting()` que imprima `"Hello friend!"`:

```kotlin
fun greeting() {
  println("Hello friend!")
}
```

Una función solo se ejecutará cuando sea **llamada** o **invocada**. Si bien la función `greeting()` existirá fuera de la función `main()`, llamaremos a `greeting()` dentro de `main()` escribiendo el nombre de la función seguido de paréntesis:

```kotlin
fun greeting() {
  println("Hello friend!")
}

fun main() {
  // Invocar la función
  greeting()
}
// Imprime: Hello friend!
```

Cada vez que llamemos a `greeting()`, se mostrará `"Hello friend!"` en la terminal.

---

## Arguments

Los **argumentos** son piezas de datos que podemos proporcionar a nuestras funciones para producir resultados dinámicos. Podemos incluir tantos argumentos como la función necesite.

Piensa en la función `println()`. Cuando la usamos, le damos un solo argumento: un valor `String`. Ese valor se pasa y se muestra en la terminal.

Para pasar argumentos a una función, necesitamos agregar **parámetros** a nuestro encabezado de función. Los parámetros son los nombres que se les dan a los datos que se pasan a una función. Por ejemplo:

```kotlin
fun calculateForce(mass: Int, acceleration: Int) {
  var force = mass * acceleration
  println("The force is $force Newtons.")
}
```

En los paréntesis del encabezado de la función, agregamos dos parámetros: `mass` y `acceleration`.

- Cada parámetro está separado por una coma (`,`).
- El parámetro comienza con su nombre, seguido de dos puntos (`:`) y luego el tipo de parámetro.
- Los parámetros se pueden referenciar dentro del cuerpo de la función como una variable.

El valor de cada parámetro está determinado por los argumentos utilizados al invocar la función:

```kotlin
calculateForce(5, 12)
```

Cuando se llama a `calculateForce()`, los valores `5` y `12` se pasan como argumentos a la función. El valor del parámetro `mass` será `5` y el valor de `acceleration` será `12`. Invocar la función con estos valores de argumento nos dará la siguiente salida:

```
The force of this object is 60 Newtons.
```

Un concepto importante que entender sobre las funciones es el **alcance (scope)**. El alcance dicta el entorno en el que se puede acceder a una variable dentro de un programa. Si intentáramos referenciar la variable `force` en cualquier parte de nuestro programa fuera de la función `calculateForce()`, recibiríamos un error porque la variable solo existe dentro de esa función.

----

## Named and Default Arguments

Podemos ir más allá con los argumentos, dándoles nombre o estableciendo valores predeterminados para los parámetros.

Para mejorar la legibilidad de nuestro código, podemos **nombrar nuestros argumentos** al invocar la función. Por ejemplo, con nuestra función `calculateForce()`, podemos agregar los nombres de los argumentos a nuestra llamada de función:

```kotlin
fun calculateForce(mass: Int, acceleration: Int) {
  var force = mass * acceleration
  println("The force is $force Newtons.")
}

fun main() {
  // Invocar la función con argumentos nombrados.
  calculateForce(acceleration = 12, mass = 5) 
  // Imprime: The force is 60 Newtons.
}
```

Ten en cuenta que al nombrar nuestros argumentos cuando invocamos una función, no necesitamos colocar los argumentos en el mismo orden en que aparecen en el encabezado de la función.

Los **argumentos predeterminados** proporcionan valores a los argumentos de la función si no se proporciona ninguno cuando se llama a la función. Para implementar esto, agregamos un operador de asignación (`=`) y un valor después del argumento en el encabezado de la función, así:

```kotlin
fun greetCustomer(name: String = "Customer") {
  println("Hello, $name.")
}
```

En la función `greetCustomer()`, establecemos el valor predeterminado del argumento `name` como `"Customer"`.

Si invocamos esta función con un valor `String`, ese valor se convertirá en el valor de `name`; sin embargo, si no se da ningún argumento, `name` tendrá el valor de `"Customer"`:

```kotlin
greetCustomer("Cynara") // Imprime: Hello, Cynara.
greetCustomer()         // Imprime: Hello, Customer.
```

----

## Return Statements

Las funciones de los ejercicios anteriores producían salida mediante instrucciones `println()` contenidas en su cuerpo de código; sin embargo, también tenemos la capacidad de **devolver un valor** desde una función.

Una declaración `return` devuelve un valor de una función y lo pasa de vuelta al lugar donde se invocó la función. Este valor puede luego ser utilizado en todo nuestro programa.

Si queremos que una función devuelva un valor, debemos agregar el **tipo de retorno** al encabezado de la función, así como una declaración `return` en el cuerpo. Observa el siguiente ejemplo, donde creamos una función que toma una lista de `Int` y devuelve un valor `Int`:

```kotlin
fun listSum(myList: List<Int>): Int {
  var sum = 0
  // Iterar a través de cada valor y sumarlo
  for (i in myList) {
    sum += i
  }
  // Declaración de retorno
  return sum
}
```

- El **tipo de retorno** describe el tipo de dato del valor que se devuelve y se indica después de los paréntesis y dos puntos `:` en el encabezado de la función.
- La palabra clave `return` se usa para declarar una declaración de retorno. En este caso, `sum` es el valor que se devuelve.
- Cualquier línea de código que exista después de la declaración `return` no se ejecutará.

Anteriormente, invocábamos funciones llamándolas en su propia línea de código; sin embargo, también podemos invocar una función y asignarla a una variable para usarla más tarde.

Por ejemplo, podemos asignar `total` al valor de retorno de `listSum()` después de enviarle una lista de valores `Int` (`billsToPay`):

```kotlin
var billsToPay = mutableListOf(44, 29, 104, 79)

// Asignar total al valor de retorno de listSum().
var total = listSum(billsToPay)
println("Your bill total is $total dollars.")
```

Esto mostrará lo siguiente:

```
Your bill total is 256 dollars.
```

----

## Single Expression Functions

Si una de nuestras funciones contiene solo una única expresión, podemos escribir la función con una sintaxis abreviada. Esta sintaxis nos permite crear una función usando solo una línea de código.

Por ejemplo, supongamos que creamos una función que devuelve un número dado (`num`) elevado a la potencia de 2. Usando la sintaxis de función normal, nuestro código se vería así:

```kotlin
fun powerOf2(num: Int): Int {
  return num * num
}
```

Usando la sintaxis de función de expresión única, podemos reescribir el código anterior eliminando las llaves (`{` y `}`) así como la palabra clave `return`:

```kotlin
fun powerOf2(num: Int): Int = num * num
```

Observa cómo el valor de retorno (`num * num`) va precedido de un operador de asignación (`=`).

Podemos acortar aún más este código eliminando el tipo de retorno. Usando la inferencia de tipos, el compilador puede deducir el tipo del valor de retorno, haciendo que nuestra función se vea así:

```kotlin
fun powerOf2(num: Int) = num * num
```

----

## Function Literals

Para simplificar cómo definimos funciones, podemos usar **literales de función**. Un literal de función es una función sin nombre que puede tratarse como un valor: podemos llamarlas, asignarlas como variables, pasarlas como argumentos y devolverlas desde una función, como haríamos con cualquier otro valor.

Veremos dos tipos de literales de función: **funciones anónimas** y **expresiones lambda**. Si bien las funciones anónimas y las expresiones lambda tienen el mismo propósito, usan una sintaxis ligeramente diferente.

Para empezar, creemos una función anónima almacenada en una variable llamada `quotient` que devuelve el cociente de dos números:

```kotlin
val quotient = fun(num1: Int, num2: Int): Double { 
  return num1 / num2 
}
```

- La función anónima está contenida en una variable llamada `quotient`.
- La palabra clave `fun` se coloca después del operador de asignación y no va seguida de un nombre (de ahí lo de "anónima").
- `quotient` tiene un **tipo de función** de `(Int, Int) -> Double`.
- El tipo de función describe el tipo de argumento y el tipo de retorno de la función anónima. Los argumentos están contenidos entre paréntesis, mientras que el tipo de retorno se indica después de `->`. En este caso, la función anónima toma dos `Int` y devuelve un único `Double`.

Para llamar a esta función anónima, simplemente llamaremos a `quotient` como lo haríamos con cualquier otra variable:

```kotlin
println(quotient(10, 5)) // Imprime: 2
```

Una **expresión lambda** es muy similar a una función anónima; sin embargo, su sintaxis concisa hace que las expresiones lambda sean una opción más popular entre los programadores. Se sugiere usar una expresión lambda en lugar de una función anónima cuando se crea una función con una sola expresión.

Si recreáramos nuestra variable `quotient` usando una expresión lambda, nuestro código se vería así:

```kotlin
val quotient = { num1: Int, num2: Int -> num1 / num2 }

println(quotient(10, 5)) // Imprime: 2
```

- La expresión lambda está contenida entre llaves: `{` y `}`.
- Indicamos los nombres de los parámetros, así como sus tipos de datos.
- El valor de retorno `num1 / num2` se coloca después del símbolo `->`.
- Incluir el tipo de retorno es opcional porque el compilador puede usar la inferencia de tipos para deducir el tipo de dato del valor de retorno.

Para obtener más información sobre funciones anónimas y expresiones lambda, consulta la Guía de Lenguaje de Kotlin.

-----

## Review

Felicitaciones por completar esta lección. Repasemos lo que aprendimos:

- Una **función** es un bloque de código reutilizable que puede ser invocado a lo largo de un programa.
- Para ejecutar una función, debe ser llamada en algún lugar del programa.
- Un **argumento** es un dato que se le proporciona a la función.
- Los argumentos tienen la opción de ser nombrados y/o recibir valores predeterminados.
- Una declaración `return` devuelve un dato desde la función hacia el lugar donde fue llamada.
- Las funciones con una sola expresión pueden escribirse con una sola línea de código usando sintaxis abreviada.
- Un **literal de función** es una función sin nombre que puede ser pasada como expresión.

-----

## Resumen en 5 líneas

1. Declaras con `fun nombre(param: Tipo): TipoRetorno { ... }` y la ejecutas llamándola: `nombre(valor)`.
2. Los argumentos pueden ser nombrados (`saludar(name = "Ana")`) y tener valores por defecto.
3. `return` devuelve un valor a quien llamó; el código después de `return` no se ejecuta.
4. Una variable declarada dentro de una función solo existe ahí (alcance).
5. Una función de una expresión se escribe `fun cuadrado(n: Int) = n * n`; una lambda es una función sin nombre entre llaves.

-----

## Siguiente lección

Sigue con: [Recursión](02-recursion.md).
