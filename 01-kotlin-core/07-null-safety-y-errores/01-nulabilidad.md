# Nulabilidad y llamados seguros en Kotlin

## ¿Qué es la nulabilidad?

La **nulabilidad** significa que una variable puede tener un valor o puede no tener nada (`null`).

En Kotlin, una variable normal **no acepta `null`**:

```kotlin
val emailObligatorio: String = "user@email.com"
```

Si quieres permitir que una variable pueda ser nula, debes agregar `?` despu?s del tipo:

```kotlin
val emailOpcional: String? = null
```

La diferencia es simple:

- `String`: siempre debe tener un texto.
- `String?`: puede tener un texto o puede ser `null`.

Esto importa porque Kotlin te obliga a manejar los valores nulos antes de usarlos. Esa es la gracia: evitar errores t?picos como `NullPointerException`.

-----

## Acceder a una variable nulable de forma segura

Si una variable puede ser `null`, Kotlin no te deja acceder directamente a sus propiedades:

```kotlin
val emailOpcional: String? = null

// emailOpcional.length // Error
```

?Por qu?? Porque si `emailOpcional` es `null`, no existe ning?n texto del cual obtener `length`.

Para resolverlo, se usa el **safe call** `?.`:

```kotlin
val longitudEmail = emailOpcional?.length

println("Longitud del correo: $longitudEmail") // Imprime: null
```

`?.` significa:

> ?Accede a esta propiedad solo si el valor no es null. Si es null, devuelve null sin romper el programa?.

-----

## El operador Elvis `?:`

A veces no quieres mostrar `null`, porque no es ?til para el usuario. Para eso existe el **operador Elvis** `?:`.

Sirve para definir un valor por defecto cuando algo es `null`:

```kotlin
val emailOpcional: String? = null

val longitudEmail = emailOpcional?.length ?: 0

println(longitudEmail) // Imprime: 0
```

Se lee as?:

> ?Si `emailOpcional?.length` tiene valor, ?salo. Si es `null`, usa `0`?.

Ejemplos comunes:

```kotlin
val nombre: String? = null

val nombreFinal = nombre ?: "an?nimo"
val textoFinal = nombre ?: ""
val longitud = nombre?.length ?: 0
```

-----

## Cuidado con `!!`

Kotlin tambi?n tiene el operador `!!`, llamado **non-null assertion**.

Este operador fuerza a Kotlin a tratar una variable nulable como si no fuera nula:

```kotlin
val emailOpcional: String? = null

println(emailOpcional!!.length) // Lanza NullPointerException
```

Esto es peligroso. Si el valor realmente es `null`, el programa se rompe.

Regla pr?ctica: usa `!!` solo si est?s completamente seguro de que el valor existe. Y aun as?, la mayor?a de veces es mejor usar `?.` o `?:`.

-----

## Ejemplo práctico

Supongamos que tienes un email obligatorio y un nombre opcional:

```kotlin
val email: String = "user@email.com"
val nombre: String? = null

val nombreFinal = nombre ?: "an?nimo"
val longitudNombre = nombre?.length ?: 0

println("Email: $email, Usuario: $nombreFinal, Longitud: $longitudNombre")
```

Salida:

```text
Email: user@email.com, Usuario: an?nimo, Longitud: 0
```

El programa no muestra `null`, no se rompe y comunica una respuesta clara.

-----

## Resumen

- `null` representa ausencia de valor.
- `String` no acepta `null`.
- `String?` s? acepta `null`.
- `?.` permite acceder de forma segura a una propiedad.
- `?:` da un valor por defecto si el resultado es `null`.
- `!!` fuerza el acceso, pero puede lanzar `NullPointerException`.

La idea importante es esta: no escribas c?digo esperando que el dato exista m?gicamente. Si un valor puede faltar, tu c?digo debe decir qu? hacer en ese caso.


## Caso práctico: Lectura de datos del usuario con `readLine`

`readLine()` sirve para leer texto que el usuario escribe en la consola.

Primero mostramos un mensaje con `print()` para pedir el dato:

```kotlin
print("Ingresa tu nombre: ")
val nombre = readLine()

println("Hola, $nombre")
```

Cuando el programa llega a `readLine()`, se queda esperando hasta que el usuario escriba algo y presione **Enter**.

### `readLine()` puede devolver `null`

En Kotlin, `readLine()` devuelve un `String?`.

Eso significa que puede devolver:

- un texto escrito por el usuario;
- una cadena vacía si el usuario solo presiona **Enter**;
- `null` si la entrada termina inesperadamente.

Por eso conviene usar el operador Elvis `?:` para dar un valor por defecto:

```kotlin
print("Ingresa tu email: ")
val email = readLine() ?: "sin email"

println("Email: $email")
```

Si `readLine()` devuelve `null`, la variable `email` toma el valor `"sin email"`.

### Limpiar espacios con `trim()`

Muchas veces el usuario escribe espacios de más. Para quitarlos al inicio y al final usamos `trim()`:

```kotlin
print("Ingresa tu mensaje: ")
val mensaje = readLine()?.trim() ?: ""

println("Mensaje: $mensaje")
```

Acá pasan dos cosas importantes:

- `?.trim()` ejecuta `trim()` solo si el valor no es `null`;
- `?: ""` asegura que `mensaje` siempre tenga un texto válido.

### Ejemplo completo

```kotlin
fun main() {
    print("Ingresa tu email: ")
    val email = readLine()?.trim() ?: "sin email"

    print("Ingresa el destinatario: ")
    val destinatario = readLine()?.trim() ?: "sin destinatario"

    print("Ingresa el asunto: ")
    val asunto = readLine()?.trim() ?: "sin asunto"

    print("Ingresa el mensaje: ")
    val mensaje = readLine()?.trim() ?: ""

    println()
    println("Vista previa del email")
    println("De: $email")
    println("Para: $destinatario")
    println("Asunto: $asunto")
    println("Mensaje: $mensaje")
}
```

### Idea clave

`readLine()` permite que el programa reciba datos reales del usuario. Como puede devolver `null`, lo correcto es manejar ese caso con `?:` o con llamadas seguras como `?.`.

-----