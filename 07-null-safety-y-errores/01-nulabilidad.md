# Nulabilidad y llamados seguros en Kotlin

## En una frase

En Kotlin una variable no puede ser `null` salvo que lo declares con `?`, y el compilador te obliga a manejar ese caso antes de usarla.

-----

## Antes de empezar

Conviene que ya sepas:

* Tipos de datos: [Tipos y variables](../02-tipos-y-variables/01-tipos-y-variables.md).
* `readLine()` y funciones de `String`: [Texto](../02-tipos-y-variables/02-texto.md).

Palabras nuevas (también están en el [Glosario](Glosario.md)):

* **Aserción no nula (`!!`):** Operador que obliga a tratar un valor nulable como no nulo. Si el valor es `null`, lanza `NullPointerException`.
* **Llamada segura (`?.`):** Accede a una propiedad o función solo si el valor no es `null`; si lo es, devuelve `null`.
* **`null`:** Ausencia de valor.
* **NullPointerException:** Error que ocurre al intentar usar un valor `null` como si existiera.
* **Operador Elvis (`?:`):** Devuelve el valor de la izquierda si no es `null`, o el de la derecha si lo es.
* **Tipo nulable:** Tipo terminado en `?` (como `String?`) que acepta `null`.

-----

## ¿Qué es la nulabilidad?

La **nulabilidad** significa que una variable puede tener un valor o puede no tener nada (`null`).

En Kotlin, una variable normal **no acepta `null`**:

```kotlin
val emailObligatorio: String = "user@email.com"
```

Si quieres permitir que una variable pueda ser nula, debes agregar `?` después del tipo:

```kotlin
val emailOpcional: String? = null
```

La diferencia es simple:

- `String`: siempre debe tener un texto.
- `String?`: puede tener un texto o puede ser `null`.

Esto importa porque Kotlin te obliga a manejar los valores nulos antes de usarlos. Esa es la gracia: evitar errores típicos como `NullPointerException`.

-----

## Acceder a una variable nulable de forma segura

Si una variable puede ser `null`, Kotlin no te deja acceder directamente a sus propiedades:

```kotlin
val emailOpcional: String? = null

// emailOpcional.length // Error
```

¿Por qué? Porque si `emailOpcional` es `null`, no existe ningún texto del cual obtener `length`.

Para resolverlo, se usa el **safe call** `?.`:

```kotlin
val longitudEmail = emailOpcional?.length

println("Longitud del correo: $longitudEmail") // Imprime: null
```

`?.` significa:

> "Accede a esta propiedad solo si el valor no es null. Si es null, devuelve null sin romper el programa".

-----

## El operador Elvis `?:`

A veces no quieres mostrar `null`, porque no es útil para el usuario. Para eso existe el **operador Elvis** `?:`.

Sirve para definir un valor por defecto cuando algo es `null`:

```kotlin
val emailOpcional: String? = null

val longitudEmail = emailOpcional?.length ?: 0

println(longitudEmail) // Imprime: 0
```

Se lee así:

> "Si `emailOpcional?.length` tiene valor, úsalo. Si es `null`, usa `0`".

Ejemplos comunes:

```kotlin
val nombre: String? = null

val nombreFinal = nombre ?: "anónimo"
val textoFinal = nombre ?: ""
val longitud = nombre?.length ?: 0
```

-----

## Cuidado con `!!`

Kotlin también tiene el operador `!!`, llamado **non-null assertion**.

Este operador fuerza a Kotlin a tratar una variable nulable como si no fuera nula:

```kotlin
val emailOpcional: String? = null

println(emailOpcional!!.length) // Lanza NullPointerException
```

Esto es peligroso. Si el valor realmente es `null`, el programa se rompe.

Regla práctica: usa `!!` solo si estás completamente seguro de que el valor existe. Y aun así, la mayoría de las veces es mejor usar `?.` o `?:`.

-----

## Ejemplo práctico

Supongamos que tienes un email obligatorio y un nombre opcional:

```kotlin
val email: String = "user@email.com"
val nombre: String? = null

val nombreFinal = nombre ?: "anónimo"
val longitudNombre = nombre?.length ?: 0

println("Email: $email, Usuario: $nombreFinal, Longitud: $longitudNombre")
```

Salida:

```text
Email: user@email.com, Usuario: anónimo, Longitud: 0
```

El programa no muestra `null`, no se rompe y comunica una respuesta clara.

-----

## Resumen

- `null` representa ausencia de valor.
- `String` no acepta `null`.
- `String?` sí acepta `null`.
- `?.` permite acceder de forma segura a una propiedad.
- `?:` da un valor por defecto si el resultado es `null`.
- `!!` fuerza el acceso, pero puede lanzar `NullPointerException`.

La idea importante es esta: no escribas código esperando que el dato exista mágicamente. Si un valor puede faltar, tu código debe decir qué hacer en ese caso.


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

## Resumen en 5 líneas

1. `String` nunca es `null`; `String?` sí puede serlo.
2. `?.` accede a una propiedad o función solo si el valor no es `null`; si lo es, devuelve `null`.
3. `?:` (Elvis) da un valor por defecto cuando lo de la izquierda es `null`.
4. `!!` fuerza el acceso y lanza `NullPointerException` si el valor es `null`: úsalo casi nunca.
5. `readLine()` devuelve `String?`, por eso se combina con `?.trim()` y `?:`.

-----

## Siguiente lección

Sigue con: [Excepciones](02-excepciones.md).
