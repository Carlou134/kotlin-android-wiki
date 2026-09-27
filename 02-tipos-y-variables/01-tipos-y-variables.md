# Data Types & Variables

## En una frase

Una variable es un nombre que guarda un valor de un tipo fijo; si la declaras con `val` no puede cambiar, y con `var` sí.

-----

## Antes de empezar

Conviene que ya sepas:

* `main()` y `println()`: [Introducción a Kotlin](../01-fundamentos/01-introduccion.md).

Palabras nuevas (también están en el [Glosario](Glosario.md)):

* **camelCase:** Convención de nombres: primera palabra en minúscula y las siguientes con mayúscula inicial, como `guitarName`.
* **Inferencia de tipos:** Capacidad del compilador de deducir el tipo de una variable a partir de su valor, sin que lo escribas.
* **Inmutable (`val`):** Variable cuyo valor no puede reasignarse después de inicializarla.
* **Mutable (`var`):** Variable cuyo valor puede cambiar a lo largo del programa.
* **Tipo de dato:** Qué clase de valor guarda una variable (`String`, `Int`, `Boolean`...) y qué operaciones admite.
* **Variable:** Nombre que guarda un valor para usarlo más adelante en el programa.

-----

## Introduction

En esta lección, exploraremos los componentes fundamentales de un lenguaje de programación: los **tipos de datos** y las **variables**. Un tipo de dato representa el tipo de valor que posee un objeto. Estos son algunos de los tipos de datos admitidos en Kotlin:

- **String** (cadena): una secuencia de caracteres encerrada entre comillas dobles.
- **Integer** (Int) (entero): números enteros y sus opuestos.
- **Boolean** (booleano): valores verdaderos o falsos (`true` o `false`).

Cada variable en un programa Kotlin poseerá uno de estos tipos de datos. Piensa en una variable como un contenedor virtual único que contiene cierta información que luego puede ser utilizada y referenciada a lo largo de un programa.

Observa la animación de la derecha. Cada elemento representa una variable que contiene un tipo específico de dato dependiendo de su valor. Codey mete en su mochila un `lunchBox` (lonchera) que contiene el valor `"sandwich"` y una `waterBottle` (botella de agua) con 32 onzas de agua. `tentPacked` (carpa empacada) contiene un valor booleano que representa si la carpa está en la mochila de Codey. Con estos valores, podemos escribir un programa que verifique si Codey tiene todo lo que necesita para su viaje.

Este es solo uno de los muchos programas que podemos construir con estas variables. Cada variable tendría su propio lugar y propósito en un programa, y sus respectivos tipos de datos determinan cómo podemos interactuar con ellas.

![Animación de variables](../Images/Kotlin_VariablesBackpack_Animation_2.gif)

-----

## Declaring a Mutable Variable


Ahora que nos hemos familiarizado con el concepto de variables, aprendamos cómo crear una variable en Kotlin. Una declaración e inicialización completa de una variable tiene la siguiente forma:

```kotlin
var nombreVariable: Tipo = valor
```

### **Declaración:**

- La palabra clave `var` especifica el comienzo de una declaración de variable. Indica una variable mutable, lo que significa que su valor puede cambiar a lo largo del programa.
- Después de la palabra clave viene el nombre de la variable en formato **camelCase**. El camelCase se refiere a una convención de nomenclatura donde la primera palabra se escribe en minúscula, la primera letra de la segunda palabra se escribe en mayúscula, y así sucesivamente. Es importante crear nombres de variables concisos pero descriptivos para facilitar la legibilidad y el mantenimiento. Un famoso ingeniero de software dijo una vez:

> *"…Literalmente pasaré horas con gusto en los nombres de identificadores: nombres de variables, nombres de métodos, etc., para que mi código sea legible…"* — Joshua Bloch

- El nombre de la variable va seguido del tipo de dato que se pretende almacenar. El tipo siempre debe escribirse con mayúscula inicial.

### **Inicialización:**

- Una vez que la variable ha sido declarada, podemos inicializarla con un valor. El signo `=`, conocido como operador de asignación en programación, asigna un valor a la variable.

Apliquemos esta sintaxis a nuestra primera variable en Kotlin:

```kotlin
var guitarName: String = "Fender Stratocaster"
```

La variable `guitarName` ha sido asignada a una posición en la memoria por el compilador y ahora puede ser referenciada y utilizada en todo nuestro programa.

El compilador también reconoce que esta variable es mutable debido a la palabra clave `var` delante del nombre de la variable; exploremos qué significa eso.

A veces puede ser necesario declarar una variable en nuestro programa, sabiendo que se usará más adelante, pero sin tener todavía un valor para inicializarla. Podemos hacerlo de la siguiente manera:

```kotlin
var nombreVariable: Tipo
```

Supongamos que estamos construyendo una aplicación musical que escuchará las notas tocadas en una guitarra. Podemos declarar una variable `notePlayed` e inicializarla con un valor cuando el usuario toque una nota:

```kotlin
var notePlayed: String  // declaración

notePlayed = "B"        // inicialización

println(notePlayed)     // Imprime: B
```

Observa cómo no necesitamos usar la palabra clave `var` en la segunda línea, ya que solo se usa en la declaración inicial. También hemos omitido el tipo de dato, ya que esta información solo necesita especificarse una vez.

-----

## Declaring an Immutable Variable

Otro tipo de variable que existe en Kotlin es la variable inmutable. El término *inmutable* significa una variable cuyo valor no puede cambiar después de su inicialización.

Las variables inmutables se indican con la palabra clave `val` y a menudo se denominan constantes en algunos lenguajes. Así es como se declaran en Kotlin:

```kotlin
val nombreVariable: Tipo = valor
```

Veamos un ejemplo. La siguiente variable contiene el valor de la proporción áurea (Golden Ratio):

```kotlin
val goldenRatio: Double = 1.618
```

Dado que sabemos que este valor se mantiene constante, es mejor declararlo con la palabra clave `val` en lugar de `var`, ya que no tenemos intención de cambiarlo. Aunque una declaración con `var` también habría sido aceptada por el compilador, la estrategia preferida es declarar variables con `var` solo si estamos seguros de que el valor cambiará en algún punto del programa; de lo contrario, usar siempre `val`.

Veamos qué sucede cuando intentamos reasignar `goldenRatio` a otro valor:

```kotlin
goldenRatio = 3.2
```

La línea de código anterior daría como resultado el siguiente error en la terminal, indicando que no se permite la reasignación:

```
Pi.kt:5:3: error: val cannot be reassigned
  goldenRatio = 3.2
  ^
```

----

## Type Inference

En Kotlin, podemos ahorrar pulsaciones de teclado y optimizar nuestras declaraciones de variables con una característica conocida como **inferencia de tipos**.

La inferencia de tipos indica que el compilador puede deducir el tipo de una variable declarada, y su tipo de dato puede omitirse en la declaración. Observa la siguiente declaración de variable:

```kotlin
var lightsOn: Boolean = true
```

Estamos indicando explícitamente el tipo de variable, pero el compilador de Kotlin también es lo suficientemente inteligente como para hacer esta suposición incluso sin la presencia de un tipo:

```kotlin
var lightsOn = true  // declaración válida
```

El compilador reconoce que un valor `true` o `false` almacenado en una variable corresponde al tipo de dato `Boolean`, por lo tanto, la variable `lightsOn` es un `Boolean`.

Es importante saber que, independientemente de cómo se declare una variable, su tipo no puede cambiar a lo largo de un programa. Una variable `Boolean`, ya sea declarada explícitamente o inferida, solo puede contener un valor `true` o `false`.

Si intentamos asignar un valor que no sea `Boolean`:

```kotlin
lightsOn = "no"  // error
```

El compilador arrojará el siguiente error:

```
Inference.kt:4:14: error: type mismatch: inferred type is String but Boolean was expected
  lightsOn = "no"
```

**Nota:** Las declaraciones de variables con tipos de datos especificados explícitamente y aquellas con tipos de datos inferidos son ambas formas válidas de crear variables. El compilador aceptará cualquiera de las dos sintaxis, por lo que elegir una u otra es simplemente cuestión de preferencia del desarrollador.

-----


## Tipos básicos más usados

En Kotlin, el tipo de dato define qué clase de valor puede guardar una variable y que operaciones puedes hacer con ella.

```kotlin
val nombreUsuario: String = "Juan"   // texto
val emailsNoLeidos: Int = 5           // numero entero
val porcentajeLeido: Float = 75.5F    // numero decimal simple
val esUrgente: Boolean = true         // verdadero o falso
```

Usa esta idea simple:

- `String`: para texto, como nombres, mensajes o asuntos de correo.
- `Int`: para números enteros, como contadores o cantidades.
- `Float`: para decimales, como porcentajes o medidas. En Kotlin termina con `F`.
- `Boolean`: para estados lógicos: `true` o `false`.

También existen tipos con más capacidad, como `Long` para enteros muy grandes y `Double` para decimales con mayor precisión.

-----

## Mostrar variables dentro de un texto

Para incluir una variable dentro de un mensaje, Kotlin usa `$` dentro de un `String`. A esto se le llama **string template**.

```kotlin
val nombreUsuario = "Juan"
val emailsNoLeidos = 3

println("Usuario: $nombreUsuario, emails no leídos: $emailsNoLeidos")
```

Esto es más claro que unir textos con `+`, sobre todo cuando el mensaje combina varios valores.

-----

## Convertir tipos de datos

A veces un dato llega como texto, pero necesitas usarlo como número. Para eso Kotlin ofrece conversiones directas:

```kotlin
val numero = 20
val texto = "15"

val numeroComoTexto = numero.toString() // Int -> String
val textoComoNumero = texto.toInt()     // String -> Int
val numeroDecimal = numero.toFloat()    // Int -> Float
```

Un detalle importante: si divides dos `Int`, Kotlin devuelve otro `Int` y descarta los decimales.

```kotlin
val leidos = 15
val totales = 20

println(leidos / totales) // Imprime: 0
```

Para obtener `0.75`, convertir al menos uno de los valores a `Float`:

```kotlin
println(leidos.toFloat() / totales) // Imprime: 0.75
```

-----

## Ejemplo práctico: porcentaje de correos enviados

Este ejemplo combina `val`, `var`, tipos de datos, conversión y string templates:

```kotlin
val nombre = "María"
var recibidos = 6
var enviados = 4

val porcentajeEnviados = (enviados.toFloat() / (recibidos + enviados)) * 100

println("$nombre tiene enviados $porcentajeEnviados% de sus correos")
```

Por que usamos `toFloat()`? Porque si dividimos enteros, Kotlin puede perder los decimales. Al convertir `enviados` a `Float`, el cálculo mantiene la precisión necesaria para obtener el porcentaje correcto.

-----

## Review

¡Excelente trabajo! Has completado la primera lección de la sección de **Tipos de Datos** y **Variables**. Repasemos lo que hemos visto.

- Una declaración de variable consiste en una palabra clave `val` o `var` seguida de un nombre, dos puntos y un tipo de dato.
- Una inicialización de variable ocurre cuando se asigna un valor a una variable usando el operador de asignación (`=`).
- Una declaración e inicialización de variable pueden ocurrir simultáneamente en una sola línea o por separado.
- Una variable mutable se indica con la palabra clave `var` y representa un valor que se espera que cambie a lo largo de un programa.
- Una variable inmutable se indica con la palabra clave `val` y representa un valor constante.
- Kotlin puede inferir el tipo de una variable mediante la inferencia de tipos.

Siéntete libre de usar el archivo vacío `Review.kt` y la terminal de salida de la derecha para perfeccionar tu comprensión de las variables y los tipos de datos, y para practicar la escritura de código en Kotlin.

-----

## Resumen en 5 líneas

1. Una variable se declara con `val` o `var`, un nombre en camelCase, `:` y un tipo: `var nombre: String = "Ana"`.
2. `var` permite reasignar el valor; `val` no. Usa `val` por defecto y `var` solo si el valor va a cambiar.
3. Gracias a la inferencia de tipos puedes omitir el tipo: `val activo = true` es un `Boolean`.
4. El tipo de una variable nunca cambia: asignarle un `String` a una variable `Boolean` es un error de compilación.
5. Para convertir entre tipos usas funciones como `toInt()` o `toFloat()`, por ejemplo para no perder decimales al dividir.

-----

## Siguiente lección

Sigue con: [Texto: String y Char](02-texto.md).
