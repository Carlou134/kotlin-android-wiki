# Manejo de excepciones con Try Catch Finally en Kotlin

Una **excepción** es un error que ocurre mientras el programa se está ejecutando.

Si no la manejás, el programa puede detenerse de golpe.

Ejemplos comunes:

- acceder a una posición inexistente de una lista;
- dividir por cero;
- usar un valor `null` de forma incorrecta;
- recibir datos inválidos del usuario.

El manejo de excepciones sirve para responder a esos errores de forma controlada.

## `try`, `catch` y `finally`

Kotlin usa tres partes principales:

- `try`: contiene el código que podría fallar;
- `catch`: captura el error y decide qué hacer;
- `finally`: se ejecuta siempre, haya error o no.

```kotlin
val emails = listOf("ana@mail.com", "juan@mail.com")

try {
  println(emails[10])
} catch (e: IndexOutOfBoundsException) {
  println("Índice de emails no válido")
} finally {
  println("Finalizó el procesamiento de correos")
}
```

En este ejemplo, `emails[10]` falla porque la lista no tiene una posición `10`.

Pero el programa no se rompe sin control. Entra al `catch`, muestra un mensaje claro y después ejecuta el `finally`.

## ¿Qué va dentro de `try`?

Dentro de `try` colocás el código riesgoso.

```kotlin
try {
  val email = emails[10]
  println(email)
}
```

No metas todo el programa dentro de `try`. Eso es mala práctica. Poné solo la parte que realmente puede fallar.

## ¿Qué hace `catch`?

`catch` captura una excepción específica.

```kotlin
catch (e: IndexOutOfBoundsException) {
  println("No existe esa posición")
}
```

La variable `e` contiene información del error.

```kotlin
catch (e: IndexOutOfBoundsException) {
  println("Error: ${e.message}")
}
```

## ¿Qué hace `finally`?

`finally` se ejecuta siempre.

Sirve para cerrar recursos, mostrar mensajes finales o limpiar algo que debe ejecutarse sí o sí.

```kotlin
finally {
  println("Proceso terminado")
}
```

## `try` como expresión

En Kotlin, `try` también puede devolver un valor.

Esto permite escribir funciones seguras.

```kotlin
fun dividir(a: Int, b: Int): Int? {
  return try {
    a / b
  } catch (e: ArithmeticException) {
    null
  }
}

fun main() {
  println(dividir(10, 2)) // 5
  println(dividir(10, 0)) // null
}
```

Si la división funciona, devuelve el resultado.

Si ocurre una división por cero, captura el error y devuelve `null`.

## Lanzar una excepción con `throw`

También podés lanzar una excepción manualmente cuando detectás un caso inválido.

```kotlin
fun validarEmail(email: String) {
  if (!email.contains("@")) {
    throw IllegalArgumentException("Email inválido: no tiene arroba")
  }
}
```

Ejemplo de uso:

```kotlin
try {
  validarEmail("correo-invalido.com")
} catch (e: IllegalArgumentException) {
  println(e.message)
}
```

## Excepciones personalizadas

Si querés representar un error propio de tu dominio, podés crear tu propia excepción.

```kotlin
class EmailInvalidException(
  message: String
) : Exception(message)
```

Ahora podés usarla así:

```kotlin
fun validarEmails(emails: List<String>) {
  for (email in emails) {
    if (!email.contains("@")) {
      throw EmailInvalidException("Email inválido, no tiene arroba: $email")
    }
  }
}
```

Y capturarla:

```kotlin
val emails = listOf("ana@mail.com", "correo-invalido.com", "juan@mail.com")

try {
  validarEmails(emails)
  println("Todos los emails son válidos")
} catch (e: EmailInvalidException) {
  println(e.message)
}
```

## Ejercicio práctico

Crear una función que divida dos números y devuelva `null` si hay división por cero.

```kotlin
fun dividirSeguro(a: Int, b: Int): Int? {
  return try {
    a / b
  } catch (e: ArithmeticException) {
    null
  }
}

fun main() {
  val resultado1 = dividirSeguro(10, 2)
  val resultado2 = dividirSeguro(10, 0)

  println(resultado1) // 5
  println(resultado2) // null
}
```

## Buenas prácticas

- Capturá excepciones específicas, no cualquier error sin pensar.
- No uses `try/catch` para ocultar problemas de diseño.
- Mostrá mensajes claros.
- Usá `finally` solo cuando realmente necesites ejecutar algo siempre.
- Validá datos antes de que el error ocurra cuando sea posible.

## Idea clave

Las excepciones no son para “tapar errores”.

Son para manejar situaciones inesperadas de forma controlada.

Buen código no es el que nunca falla. Buen código es el que sabe qué hacer cuando algo falla.

