# Classes

## Introduction to Classes

Kotlin, al igual que Java, es un lenguaje de programación orientado a objetos. El término "orientado a objetos" se refiere a un lenguaje que está compuesto por **objetos** y puede utilizar diversas metodologías como encapsulamiento, polimorfismo y herencia. En esta lección, haremos una breve introducción al mundo de los objetos con una herramienta conocida como **clases**.

Las clases nos permiten crear planos de objetos cotidianos y luego utilizar este plano para crear tantas instancias únicas como queramos. Supongamos que estamos construyendo un programa para un concesionario de automóviles. Necesitaríamos almacenar algunas de las siguientes características básicas para cada automóvil:

- Modelo
- Año
- Color

Podemos usar una clase para crear un plano con todas estas propiedades, a partir del cual luego crearíamos cada automóvil individual. Cada automóvil único sería una **instancia** de esta clase, también conocida como **objeto**.

Con cada nuevo concepto que hemos aprendido en este curso, hemos hecho que nuestros programas sean más dinámicos y personalizables, y las clases no serán diferentes. Con las clases, tenemos la capacidad de imitar objetos del mundo real en nuestro código y adaptarlos a nuestro gusto.

En esta lección, nos centraremos en crear clases, generar objetos individuales a partir de ellas y explorar sus funcionalidades disponibles.

----

## Clases en Kotlin, explicado simple

Una **clase** es un molde para crear objetos.

Sirve para agrupar en una sola estructura:

- datos, llamados **propiedades** o **atributos**;
- acciones, llamadas **funciones** o **métodos**.

Pensá en un correo electrónico. Un email puede tener asunto, remitente, mensaje y un estado para saber si fue leído o no. Todo eso se puede modelar con una clase.

```kotlin
class Email(
  val asunto: String,
  val remitente: String,
  val mensaje: String
) {
  var leido: Boolean = false

  fun marcarComoLeido() {
    leido = true
  }

  fun marcarComoNoLeido() {
    leido = false
  }

  fun mostrarInfo() {
    println("$remitente - $asunto - Leído: $leido")
  }
}
```

### Clase vs objeto

La clase es la plantilla. El objeto es una instancia concreta creada a partir de esa plantilla.

```kotlin
val email1 = Email(
  asunto = "Reunión viernes",
  remitente = "carlos@email.com",
  mensaje = "No te olvides de nuestra reunión"
)

email1.mostrarInfo()
```

`Email` es la clase. `email1` es un objeto.

Podés crear muchos objetos usando la misma clase, pero cada uno puede tener valores distintos.

### ¿Por qué `println(email1)` imprime algo raro?

Si imprimís directamente un objeto común, Kotlin muestra una representación por defecto, parecida a una referencia en memoria.

```kotlin
println(email1)
```

Por eso conviene crear un método como `mostrarInfo()` para imprimir los datos importantes del objeto.

Más adelante también existe otra opción: usar `data class`, que genera una representación más útil automáticamente.

### Clase sin constructor con parámetros

Una clase no siempre necesita recibir datos al crearse.

Por ejemplo, una bandeja de emails puede empezar vacía y luego recibir correos:

```kotlin
class BandejaEmails {
  val emails = mutableListOf<Email>()

  fun agregarEmail(email: Email) {
    emails.add(email)
  }

  fun contarNoLeidos(): Int {
    var contador = 0

    for (email in emails) {
      if (!email.leido) {
        contador += 1
      }
    }

    return contador
  }
}
```

Acá aparece una idea importante: una lista puede guardar objetos creados por vos, no solo `String`, `Int` o `Boolean`.

```kotlin
val bandeja = BandejaEmails()

bandeja.agregarEmail(email1)
println("Faltan por leer: ${bandeja.contarNoLeidos()}")
```

### Relación entre clases

Las clases pueden conectarse entre sí.

Una `BandejaEmails` puede tener muchos objetos `Email`. Un usuario podría tener muchos contactos. Una tienda podría tener muchos productos.

Eso es modelar: representar relaciones reales dentro del código.

```kotlin
val email2 = Email("Tarea", "luis@mail.com", "Mensaje 2")
val email3 = Email("Pago", "pago@mail.com", "Mensaje 3")

email1.marcarComoLeido()

val bandeja = BandejaEmails()
bandeja.agregarEmail(email1)
bandeja.agregarEmail(email2)
bandeja.agregarEmail(email3)

println("Faltan por leer: ${bandeja.contarNoLeidos()}")
```

Como `email1` fue marcado como leído, quedan dos emails sin leer.

### Ejercicio: clase `Contacto`

Un ejercicio simple para practicar es crear una clase `Contacto`.

```kotlin
class Contacto(
  val nombre: String,
  val email: String
) {
  fun mostrarContacto() {
    println("$nombre - $email")
  }
}

fun main() {
  val contacto1 = Contacto("María", "maria@mail.com")
  contacto1.mostrarContacto()
}
```

**Idea clave:** una clase no es solo “código junto”. Es una forma de representar una idea del mundo real con datos y comportamiento. Si entendés eso, la programación orientada a objetos empieza a tener sentido.

----

## Creating a Class

Ahora practiquemos la escritura de nuestras propias **clases**. Una clase en Kotlin se declara en la parte superior de un archivo Kotlin, fuera de la función `main()`, con la palabra clave `class` seguida de su nombre:

```kotlin
class Nombre {
  // cuerpo de la clase
}
```

El nombre de una clase siempre debe comenzar con una letra mayúscula y usar el formato camelCase. El **cuerpo de la clase** está contenido entre llaves y contendrá todas las propiedades y funciones relacionadas con esa clase. Analizaremos más de cerca las funciones de clase en un ejercicio posterior.

**Nota:** Es opcional que una clase de Kotlin contenga un cuerpo, por lo tanto, si la clase no tiene cuerpo, podemos omitir sus llaves, lo que da como resultado la siguiente declaración:

```kotlin
class Nombre
```

Revisitemos nuestro ejemplo del mundo real, `Car`. Supongamos que creamos la siguiente clase:

```kotlin
class Car
```

¡Nuestro plano está completo! Pero vacío no nos sirve de mucho. Cualquier automóvil posee varias propiedades, incluido el año en que se fabricó, su nombre de modelo y su color. Agreguemos estas:

```kotlin
class Car {
  val year = 2011
  val model = "Jeep"
  val color = "Blue"
}
```

Hemos creado las propiedades `year`, `model` y `color` y las hemos inicializado con valores. Ahora, cada vez que creemos un objeto `Car` a partir de esta clase, poseerá las propiedades `year`, `model` y `color` y tendrá sus respectivos valores. En el próximo ejercicio, aprenderemos cómo crear **objetos** a partir de una clase. Por ahora, practiquemos con la sintaxis de clases de Kotlin. 🚗

----

## Creating an Instance

Ahora que hemos creado nuestra primera clase, creemos una instancia de ella. Una **instancia** de una clase se asemeja a un objeto que es miembro de esa clase.

Recordemos la clase `Car` del ejercicio anterior:

```kotlin
class Car {
  val year = 2011
  val model = "Jeep"
  val color = "Blue"
}
```

Para crear una instancia de esta clase, utilizaríamos la siguiente sintaxis:

```kotlin
val myCar = Car()
```

- Hemos declarado que el nombre de nuestra instancia/objeto sea `myCar`. Este nombre es arbitrario y queda a elección del desarrollador.
- Después del operador de asignación, hemos invocado la clase `Car` con el nombre de la clase seguido de paréntesis — la misma sintaxis que usamos al invocar una función.
- Con esta línea de código, la instancia `myCar` ahora posee todas las propiedades y sus respectivos valores que se especificaron dentro del cuerpo de la clase.

Al igual que hemos hecho con propiedades integradas de Kotlin como `.length` y `.size`, podemos utilizar la **notación de punto** y agregar los nombres de las propiedades de nuestra clase a nuestra instancia/objeto para recuperar sus valores.

```kotlin
println(myCar.year)  // Imprime: 2011
println(myCar.model) // Imprime: Jeep
println(myCar.color) // Imprime: Blue
```

¡Voilà! Has creado tu primer objeto. Piensa en lo que has aprendido sobre las clases y sus propiedades. ¿Cuál es una forma en la que podríamos necesitar mejorar este código? Sigue los puntos de control a continuación para descubrirlo.

-----

## Primary Constructor

¿Buscas personalizar tus instancias? ¡Has llegado al lugar indicado! Las **clases** de Kotlin nos proporcionan algo llamado **constructor primario** que nos permite declarar propiedades e inicializarlas cuando instanciamos un objeto.

Veamos la sintaxis abreviada:

```kotlin
class Nombre(val x: String, val y: String)
```

Hemos introducido un par de paréntesis en el encabezado de la clase que definen las propiedades `x` e `y`. Anteriormente, habríamos tenido algo como esto:

```kotlin
class Nombre {
  val x = "some value"
  val y = "another value" 
}
```

Cada propiedad ya tendría su valor predeterminado establecido, pero con los constructores primarios, le damos a nuestro programa la capacidad de establecer los valores cuando se crea una instancia.

Refactorizando nuestro ejemplo de `Car` del ejercicio anterior para utilizar un constructor primario en el encabezado, obtenemos la siguiente línea de código:

```kotlin
class Car(val year: Int, val model: String, val color: String)
```

Ordenado, ¿verdad? Veámoslo en acción:

```kotlin
val myCar = Car(2011, "Jeep", "Blue") 
val friendsCar = Car(2015, "Mazda", "Red") 
```

Ahora hemos creado con éxito instancias únicas de nuestra clase `Car` y personalizado sus valores:

```kotlin
println(myCar.year)      // Imprime: 2011
println(friendsCar.year) // Imprime: 2015
```

------

## The Init Block

El constructor primario no contiene ningún código aparte de las propiedades y sus tipos de datos especificados. ¿Qué pasa si quisiéramos calcular la edad de una persona usando el año en que nació? Podemos hacer esto con un bloque `init`.

`init` es la abreviatura de *initializer* (inicializador) y nos permite agregar más lógica a nuestras clases.

Supongamos la siguiente clase, `Mascot`:

```kotlin
class Mascot(val name: String, val platform: String, val yearCreated: Int)
```

Nos gustaría usar el año en que se creó la mascota para calcular su edad. Podemos hacerlo agregando esta lógica dentro de un bloque `init`:

```kotlin
class Mascot(val name: String, val platform: String, val yearCreated: Int) {
  var age: Int
  
  init {
    age = 2020 - yearCreated
    println("$name is a $platform mascot and is $age years old. ")
  }
}
```

- Hemos creado una variable, `age`, dentro de la clase que usaremos para almacenar el valor del cálculo.
- Dentro del bloque `init`, agregamos nuestra expresión matemática seguida de una instrucción `println()` para mostrar el resultado final.

Ahora, cuando llamemos a la clase, deberíamos esperar ver una salida de tipo `String`:

```kotlin
val codey = Mascot("Codey", "Codecademy", 2018) 
// Imprime: Codey is a Codecademy mascot and is 2 years old.
```

----

## Member Functions

Ahora exploremos cómo agregar aún más funcionalidad a nuestras clases con funciones. Una función declarada dentro de una clase de Kotlin se conoce como **función miembro** (member function).

Las funciones miembro poseen las mismas cualidades y capacidades que las funciones diseñadas por el usuario que aprendiste en la lección anterior. La sintaxis es la misma también. Algunas cosas a tener en cuenta:

- Cuando se crea una instancia de una clase, el objeto tiene acceso a todas y cada una de las funciones que existen dentro del cuerpo de la clase.
- A diferencia del código dentro del bloque `init`, que se ejecuta automáticamente, el código dentro de las funciones solo se ejecutará cuando se llame a la función.

Por ejemplo:

```kotlin
class Cat(val name: String, val breed: String) {

  fun speak() {
    println("Meow!") 
  } 
}
```

Hemos creado una clase simple, `Cat`, con un constructor primario y una única función miembro. Ahora podemos crear una instancia de la clase de la siguiente manera y llamar a la función `speak()`:

```kotlin
var myCat = Cat("Garfield", "Persian")
myCat.speak() // Imprime: Meow!
```

🐈

-----

## Review

¡Excelente trabajo! En esta lección, hemos aprendido los siguientes conceptos:

- Una **clase** es un concepto de la programación orientada a objetos que se asemeja a un plano para objetos individuales. Una clase puede contener propiedades y funciones, y se define usando la palabra clave `class` seguida de un nombre y un cuerpo opcional.
- Una **instancia** es un miembro de una clase creado al llamar al nombre de la clase seguido de un par de paréntesis, `()`, y los argumentos necesarios.
- Podemos usar **notación de punto** (dot syntax) para acceder al valor de cada propiedad de la clase.
- Una clase puede tener propiedades con valores predeterminados o personalizables con la ayuda de **constructores primarios**.
- Un **constructor primario** nos permite establecer el valor de cada propiedad cuando instanciamos un objeto.
- El bloque `init` se invoca con cada creación de instancia y se usa para agregar lógica a la clase.
- Una función declarada dentro de una clase de Kotlin se conoce como **función miembro** de esa clase. Para invocar una función miembro, debemos llamar a la función sobre una instancia de la clase.

---
