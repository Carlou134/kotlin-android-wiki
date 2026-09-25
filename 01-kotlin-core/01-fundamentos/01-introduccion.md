# Introduction to Kotlin

## What is Kotlin?

**Bienvenido al mundo de la programación con Kotlin.**

Kotlin es un lenguaje de programación que, tras su lanzamiento en 2016 por JetBrains, ganó rápidamente popularidad entre los desarrolladores gracias a su sintaxis concisa, sus características adicionales de seguridad y su compatibilidad con el lenguaje de programación Java.

Al ser un lenguaje de propósito general, Kotlin se puede usar para desarrollo en Android, desarrollo web, ciencia de datos y más. En 2019, Google anunció Kotlin como su lenguaje preferido para el desarrollo en Android.

Las siguientes aplicaciones usan Kotlin en su pila tecnológica:

- Netflix
- Duolingo
- Pinterest
- ¡Y muchas más!

A lo largo de esta lección, aprenderemos sobre la sintaxis de Kotlin y cómo ejecutar un programa básico.

-----

## First Kotlin Program

¿Qué mejor manera de empezar a aprender Kotlin que ejecutando un programa en Kotlin? Antes de ejecutar nuestro primer programa, tomémonos un momento para entender nuestro entorno actual.

El código se escribe y edita en algo llamado editor de código. El editor de código para un ejercicio de Codecademy se encuentra en el medio de la pantalla. Si miramos en la parte superior del editor, veremos el nombre del archivo `Hello.kt`. La extensión `.kt` indica que el archivo contiene código Kotlin.

En nuestro editor de código, hay un pequeño programa en Kotlin:

```kotlin
fun main() {
  println("Hello, World!")
}
```

No te preocupes por entender qué significa este código todavía. Todo lo que hace este programa es imprimir el texto `"Hello, World!"` en la terminal de salida, que se encuentra en el lado derecho de nuestra pantalla. Cuando hacemos clic en **Ejecutar**, la terminal de salida muestra el resultado final de nuestro código.

Revisaremos la terminal de salida con frecuencia para asegurarnos de que nuestro código funcione como esperamos.

----

## The main() Function

Piensa en escribir código como si fuera una aventura: siempre necesitamos empezar en algún lugar.

En Kotlin, el viaje comienza en la función `main()`.

La función `main()` es el "punto de partida" de nuestro código y es la primera parte de nuestro programa que se ejecutará cuando ejecutemos nuestro programa. Estará presente en cada proyecto de Kotlin que creemos; de hecho, sin una función `main()`, el programa devolverá un error.

La sintaxis de la función `main()` es la siguiente:

```kotlin
fun main() {
  // El código irá aquí
}
```

- `fun` es una palabra clave de Kotlin que se usa para declarar una función.
- `main` es el nombre obligatorio de la función de inicio.
- Después del nombre de la función vienen un par de paréntesis vacíos `()`.
- El código que se escribe dentro de las llaves (`{ }`) se considera el "cuerpo" de una función. Cuando presionamos **Enter** para crear una nueva línea de código dentro del cuerpo, el código se indentará automáticamente:

```kotlin
fun main() {
  println("Hello, World!")
}
```

-----

## Print Statements

En el ejercicio anterior, pudimos mostrar el texto `¡Coding es divertido!` en la terminal de salida con el siguiente código:

```kotlin
fun main() {
  println("Coding is fun!")
}
```

Logramos esta hazaña usando una instrucción de impresión (print statement). Una instrucción de impresión muestra valores en la terminal.

Cuando queremos usar una instrucción de impresión en Kotlin, utilizamos la función `println()`, que imprime un valor antes de crear una nueva línea en la terminal de salida:

```kotlin
println("To infinity and beyond!")
```

Cualquier valor que queramos imprimir debe colocarse dentro de los paréntesis de la instrucción `println()`. Si queremos mostrar texto, debemos escribirlo entre comillas dobles (`""`). El código del ejemplo anterior mostrará:

```
To infinity and beyond!
```

Incluso podemos usar instrucciones de impresión para mostrar valores numéricos y el resultado de expresiones. Por ejemplo:

```kotlin
println(5 + 5)
```

Tendrá como salida:

```
10
```

A lo largo de este curso usaremos principalmente `println()`; sin embargo, existe otra opción para imprimir valores en la terminal de salida: `print()`.

La diferencia entre `println()` y `print()` es que `println()` crea una nueva línea después de mostrar un valor, mientras que `print()` no lo hace. Por ejemplo:

```kotlin
println("One")
print("Two")
print("Three")
```

Mostrará:

```
One
TwoThree
```

-----

## Order of Execution

Al igual que las instrucciones de una receta, el código se lee, compila y ejecuta en orden descendente (de arriba abajo).

Cuando ejecutamos un programa, un compilador traduce nuestro código Kotlin a algo que pueda ser leído por la Máquina Virtual de Java (JVM). La JVM es la responsable de ejecutar el código como instrucciones.

Todo esto ocurre detrás de escena entre que ejecutamos nuestro código y vemos su salida en la terminal.

Dentro de la función `main()`, la primera línea de código es la primera instrucción que lee el compilador. Por ejemplo, creemos un programa que contenga tres instrucciones de impresión:

```kotlin
fun main() {
  println("Huzzah!")
  println("Woohoo!")
  println("Yehaw!") 
}
```

En el fragmento anterior, la instrucción de impresión que contiene `"Huzzah!"` es la primera línea de código dentro de la función `main()`; por lo tanto, se imprimirá primero en la terminal de salida, seguida de `"Woohoo!"` y luego `"Yehaw!"`:

```
Huzzah!
Woohoo!
Yehaw!
```

-----

## Comments

Documentar lo que nuestro código está logrando es una buena práctica de programación porque hace que nuestro código sea comprensible tanto para el programador como para cualquier otra persona que lo lea.

Los comentarios nos permiten escribir documentación dentro de nuestros programas. Los comentarios se pueden escribir en cualquier lugar de un programa y pueden contener texto, símbolos o incluso código; todo ello será ignorado por el compilador.

Para escribir un comentario de una sola línea, escribe `//` en el programa seguido del comentario que queramos agregar:

```kotlin
// Ser o no ser: esa es la cuestión.
```

`//` simboliza el inicio de un comentario. Observa cómo el texto que aparece en un comentario tiene un color de fuente gris.

También tenemos la opción de usar un comentario multilínea, que nos permite escribir comentarios a lo largo de múltiples líneas. Los comentarios multilínea comienzan con `/*` y terminan con `*/`:

```kotlin
/*
Si nos apuñalan, ¿no sangramos?
Si nos hacen cosquillas, ¿no nos reímos?
Si nos envenenan, ¿no morimos?
Y si nos agravian, ¿no nos vengaremos?

- William Shakespeare, El mercader de Venecia
*/
```

-----

## Errors and Bugs

Aunque cada programador es diferente, hay una experiencia que todos compartimos: recibir un mensaje de error. Un error (o bug) hace que nuestros programas se comporten de manera inesperada.

Cuando recibimos un mensaje de error, casi siempre es debido a algo que escribimos incorrectamente en nuestro código. Como resultado, el compilador no puede traducir nuestro programa y devuelve un mensaje de error.

Existen muchos tipos diferentes de errores que podemos encontrar. Esto puede parecer abrumador, pero afortunadamente el compilador de Kotlin proporciona mensajes de error precisos y descriptivos, permitiéndonos localizar el problema de manera rápida y eficiente.

Por ejemplo, observa muy detenidamente el siguiente código:

```kotlin
fun main() {
  printlnn("To err is human.")
}
```

Cuando ejecutamos esto, recibimos el siguiente error:

```
Error.kt:2:3: error: unresolved reference: printlnn
  printlnn("To err is human.")
  ^
```

Este mensaje de error proporciona mucha información útil:

- `Error.kt` es el nombre del archivo que contiene el error.
- El primer número (`2`) nos indica la línea en la que se encuentra el error.
- El segundo número (`3`) nos indica el número de carácter en esa línea donde está el error.
- El texto que sigue a `error:` describe qué error estamos enfrentando.

Recibimos este error porque escribimos `printlnn` en lugar de `println` y el compilador no pudo reconocer la instrucción.

Una vez que corregimos y volvemos a ejecutar nuestro código, obtenemos la siguiente salida:

```
To err is human.
```

----

## Formatting Conventions

En programación, las convenciones de formato son las reglas sugeridas sobre cómo debe estructurarse el código en un lenguaje de programación específico. Siguiendo los estándares de formato de Kotlin, o convenciones de codificación, podemos mejorar drásticamente la legibilidad de nuestros programas.

Algunas de las convenciones de formato de Kotlin incluyen:

**Indentación:**

El código dentro de las llaves debe tener una indentación de 4 espacios. Si bien el editor de código de Codecademy generalmente usa 2 espacios, o 1 tabulación, para la indentación, asegúrate de usar 4 espacios al escribir código Kotlin fuera de la plataforma.

```kotlin
fun main() {
    // El código comienza aquí
}
```

Si no incluyéramos indentación en nuestro programa, nuestro código se ejecutaría sin errores; sin embargo, nuestro código se vería como un bloque gigante de texto, dificultando su lectura. A medida que los programas se vuelven más grandes y complicados, la indentación con fines de organización se vuelve más esencial.

**Comentarios:**

Al escribir un comentario, agrega un espacio (" ") entre `//` y el inicio del comentario:

```kotlin
// Escribe tu comentario aquí
```

**Llaves:**

Al crear la función `main()`, coloca la llave de apertura `{` en la misma línea que la construcción de la función `main()` (en este caso: `fun main()`). La llave de cierre `}` debe colocarse en su propia línea separada y debe estar alineada verticalmente con la palabra clave `fun`:

```kotlin
fun main() { 
  // El código va aquí
}
```

Ten en cuenta las convenciones anteriores al escribir código Kotlin a lo largo de este curso y fuera de la plataforma. Encontraremos más convenciones de formato a medida que avancemos en este curso. Para aprender más, consulta las convenciones de codificación en la Guía de Lenguaje de Kotlin.

----

## Review and Resources

Excelente trabajo completando la primera lección sobre aprendizaje de Kotlin. Repasemos todo el contenido que hemos visto:

- Kotlin es un lenguaje de programación de propósito general.
- La función `main()` es el punto de partida de todos los programas en Kotlin.
- Cuando queremos mostrar algo en la terminal, podemos usar instrucciones de impresión como `println()` o `print()`.
- El código se lee de arriba abajo.
- Los comentarios se usan para explicar qué está sucediendo en nuestro código.
- El lenguaje, la sintaxis y las reglas… ¡vaya! Aprender un nuevo lenguaje puede parecer abrumador; afortunadamente, hay muchos recursos en línea disponibles para nosotros sobre el lenguaje Kotlin:

  - La [**Guía de Lenguaje de Kotlin**](https://kotlinlang.org/docs/home.html) contiene la documentación oficial de Kotlin.
  - La [**Guía de Lenguaje de Kotlin sobre Formato**](https://kotlinlang.org/docs/coding-conventions.html#formatting) trata sobre las convenciones de codificación y los estándares de formato de Kotlin.
  - La [**Guía de Estilo de Kotlin**](https://developer.android.com/kotlin/style-guide?hl=es-419) contiene los estándares de codificación para Android de Google en Kotlin.
  - El **Foro de Codecademy** es un gran recurso y un lugar amigable donde los estudiantes pueden ver y hacer preguntas sobre programación.

Estas guías y recursos son invaluables para los programadores, y recomendamos encarecidamente tenerlos a mano y consultarlos con frecuencia al desarrollar en Kotlin.

-----

## ¿Por qué Kotlin?

### ¿Por qué aprender Kotlin desde cero?

Kotlin es un lenguaje de programación muy útil porque sirve para muchas cosas: hacer apps para Android, crear servicios backend, páginas web, apps para iOS y más. Es moderno, práctico y elegante.

---

### ¿Qué problemas soluciona Kotlin?

1. **Evita errores por variables vacías (null)**  
   Kotlin tiene "null safety", lo que significa que te ayuda a evitar que tu app se caiga por olvidarte de poner un valor.

2. **Reduce el código repetitivo y difícil de mantener**  
   Tiene funciones especiales (`let`, `apply`, `run`, `also`) que hacen el código más corto, limpio y fácil de entender.

---

### ¿Cómo ayuda Kotlin con datos complejos?

- **Data classes**: Sirven para representar información de forma sencilla y ordenada.  
- **Sealed classes**: Útiles para manejar modelos de datos bien definidos y seguros.

---

### ¿Cómo hacer código más claro con Kotlin?

- **`when`**: Reemplaza muchos `if` y `else`, haciendo el código más fácil de leer.  
- **Extension functions**: Permiten agregar nuevas funciones a clases existentes sin modificarlas.  
- **Higher-order functions**: Ayudan a escribir lógica compleja de forma más clara y con menos errores.

**Ejemplo práctico:** Validar un formulario de email se vuelve rápido, legible y robusto combinando operadores lógicos, null safety y funciones reutilizables.

---

### ¿Qué beneficios profesionales tiene aprender Kotlin?

- Desarrollas un pensamiento lógico y estructurado.  
- Abre muchas oportunidades laborales (hoy y en el futuro).  
- Te permite crear proyectos que impulsen tu carrera.

---

En resumen: **Kotlin es fácil de aprender, evita errores comunes, hace tu código más limpio y te prepara para trabajar en múltiples áreas de la programación.**

----