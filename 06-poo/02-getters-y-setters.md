# Getters y setters en Kotlin

## En una frase

Los getters y setters son funciones que se ejecutan al leer o al asignar una propiedad, y te permiten validar, limpiar o calcular datos.

-----

## Antes de empezar

Conviene que ya sepas:

* Propiedades y constructor: [Clases](01-clases.md).
* `if` como expresión: [Condicionales](../04-control-de-flujo/01-condicionales.md).

Palabras nuevas (también están en el [Glosario](Glosario.md)):

* **`field`:** Dentro de un getter o setter, el valor realmente guardado de la propiedad (backing field).
* **Getter:** Función que se ejecuta al leer una propiedad.
* **Propiedad calculada:** Propiedad sin valor guardado cuyo getter calcula el resultado en cada lectura.
* **Setter:** Función que se ejecuta al asignar un valor a una propiedad.

-----

Los **getters** y **setters** controlan cómo se lee y cómo se modifica una propiedad de una clase.

La idea simple es esta:

- un **getter** se ejecuta cuando leés una propiedad;
- un **setter** se ejecuta cuando cambiás el valor de una propiedad.

Sirven para agregar lógica sin repetir código: validar datos, limpiar texto, asignar valores por defecto o calcular información.

## Setter personalizado

Un setter personalizado permite controlar qué valor se guarda.

```kotlin
class Email(
  val remitente: String,
  val mensaje: String
) {
  var asunto: String = ""
    set(value) {
      field = if (value.trim().isNotEmpty()) {
        value.trim()
      } else {
        "sin asunto"
      }
    }
}
```

Acá pasan tres cosas importantes:

- `value` es el nuevo valor que alguien intenta asignar;
- `field` representa el valor real guardado internamente;
- si el asunto viene vacío, se guarda `"sin asunto"`.

Ejemplo:

```kotlin
val email1 = Email("juan@gmail.com", "Este es un mensaje")

email1.asunto = "   urgente reunión jueves   "

println(email1.asunto) // urgente reunión jueves
```

El setter limpió los espacios con `trim()` antes de guardar el valor.

## ¿Por qué se usa `field`?

Dentro de un setter no conviene hacer esto:

```kotlin
set(value) {
  asunto = value // Mal
}
```

Eso vuelve a llamar al setter una y otra vez, creando un ciclo infinito.

Por eso Kotlin usa `field`:

```kotlin
set(value) {
  field = value
}
```

`field` modifica directamente el valor interno de la propiedad.

## Getter personalizado

Un getter personalizado permite controlar qué se devuelve cuando leés una propiedad.

También puede usarse para crear una **propiedad calculada**.

```kotlin
val esImportante: Boolean
  get() = asunto.contains("urgente", ignoreCase = true)
```

Esta propiedad no guarda un valor propio. Cada vez que la leés, Kotlin calcula si `asunto` contiene la palabra `"urgente"`.

Ejemplo completo:

```kotlin
class Email(
  val remitente: String,
  val mensaje: String
) {
  var asunto: String = ""
    set(value) {
      field = if (value.trim().isNotEmpty()) {
        value.trim()
      } else {
        "sin asunto"
      }
    }

  val esImportante: Boolean
    get() = asunto.contains("urgente", ignoreCase = true)
}

fun main() {
  val email1 = Email("juan@gmail.com", "Este es un mensaje")

  println("El asunto es: ${email1.asunto}")

  email1.asunto = "urgente reunión jueves"

  println("El asunto es: ${email1.asunto}")
  println("¿El correo es importante? ${email1.esImportante}")
}
```

Salida esperada:

```text
El asunto es:
El asunto es: urgente reunión jueves
¿El correo es importante? true
```

## `if` como expresión

En Kotlin, `if` puede devolver un valor.

Por eso este código necesita `else`:

```kotlin
field = if (value.trim().isNotEmpty()) {
  value.trim()
} else {
  "sin asunto"
}
```

Como el resultado del `if` se asigna a `field`, Kotlin necesita saber qué valor usar si la condición es falsa.

## Ejercicio: clase `Contacto`

Vamos a crear una clase `Contacto` que valide su email.

Reglas:

- si el email contiene `@`, se guarda;
- si no contiene `@`, se guarda vacío;
- `esValido` devuelve `true` si el email contiene `@` y `.com`.

```kotlin
class Contacto(
  val nombre: String
) {
  var email: String = ""
    set(value) {
      field = if (value.contains("@")) {
        value
      } else {
        ""
      }
    }

  val esValido: Boolean
    get() = email.contains("@") && email.contains(".com")
}

fun main() {
  val contacto1 = Contacto("Juan")

  contacto1.email = "juan@email.com"

  println(contacto1.email)
  println(contacto1.esValido)
}
```

## Idea clave

Usá getters cuando necesitás controlar o calcular lo que se lee.

Usá setters cuando necesitás validar o transformar lo que se guarda.

No son decoración. Son una forma de proteger el estado interno de tus objetos y evitar lógica repetida por todos lados.

-----

## Resumen en 5 líneas

1. Un getter se ejecuta al leer una propiedad; un setter, al asignarle un valor.
2. En un setter personalizado, `set(value) { field = ... }` decide qué se guarda realmente.
3. Usa `field`, no el nombre de la propiedad: asignar la propiedad dentro de su propio setter lo vuelve a llamar sin fin.
4. Un getter personalizado calcula el valor en cada lectura, como `val esValido get() = email.contains("@")`.
5. Úsalos para proteger el estado del objeto en vez de repetir validaciones por todo el código.

-----

## Siguiente lección

Sigue con: [Data classes](03-data-classes.md).
