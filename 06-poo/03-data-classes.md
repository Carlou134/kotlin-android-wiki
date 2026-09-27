# Dataclases en Kotlin: ventajas sobre clases normales

## En una frase

Una `data class` es una clase pensada para guardar datos, a la que Kotlin le genera automáticamente `toString()`, `equals()`, `hashCode()` y `copy()`.

-----

## Antes de empezar

Conviene que ya sepas:

* Constructor primario y propiedades: [Clases](01-clases.md).

Palabras nuevas (también están en el [Glosario](Glosario.md)):

* **`copy()`:** Función de las data classes que crea una copia, permitiendo cambiar algunas propiedades.
* **Data class:** Clase marcada con `data` cuyo objetivo principal es guardar datos.
* **Desestructuración:** Extraer varias propiedades de un objeto en variables separadas: `val (a, b) = objeto`.
* **`hashCode()`:** Número que representa el contenido de un objeto. Lo usan colecciones como sets y maps para buscar rápido.
* **`toString()`:** Función que convierte un objeto en texto. En una data class muestra sus propiedades.

-----

Una **dataclase** en Kotlin es una clase pensada principalmente para guardar datos.

Se declara usando la palabra `data` antes de `class`.

```kotlin
data class Email(
  val asunto: String,
  val remitente: String,
  val mensaje: String
)
```

La idea es simple: si una clase solo representa información, una `data class` te evita escribir código repetitivo.

## Clase normal vs dataclase

Con una clase normal, si imprimís el objeto directamente, Kotlin puede mostrar algo poco útil:

```kotlin
class EmailNormal(
  val asunto: String,
  val remitente: String,
  val mensaje: String
)

fun main() {
  val email = EmailNormal(
    asunto = "Reunión",
    remitente = "ana@mail.com",
    mensaje = "Nos vemos el viernes"
  )

  println(email)
}
```

La salida suele verse como una referencia interna del objeto, no como sus datos.

Con una dataclase:

```kotlin
data class Email(
  val asunto: String,
  val remitente: String,
  val mensaje: String
)

fun main() {
  val email = Email(
    asunto = "Reunión",
    remitente = "ana@mail.com",
    mensaje = "Nos vemos el viernes"
  )

  println(email)
}
```

La salida es más clara:

```text
Email(asunto=Reunión, remitente=ana@mail.com, mensaje=Nos vemos el viernes)
```

Eso pasa porque una `data class` genera automáticamente un `toString()` útil.

## Ventajas principales de una dataclase

Kotlin genera automáticamente varias funciones importantes:

- `toString()`: muestra el contenido del objeto de forma legible.
- `equals()`: compara objetos por sus valores.
- `hashCode()`: genera un código hash basado en sus propiedades.
- `copy()`: crea copias modificando solo lo necesario.
- `componentN()`: permite desestructurar el objeto en variables.

No es magia. Kotlin escribe por vos código que normalmente tendrías que escribir a mano.

## Comparar dataclases con `==`

En una dataclase, `==` compara los valores de las propiedades.

```kotlin
data class Email(
  val asunto: String,
  val remitente: String,
  val mensaje: String
)

fun main() {
  val email1 = Email("Reunión", "ana@mail.com", "Nos vemos el viernes")
  val email2 = Email("Reunión", "ana@mail.com", "Nos vemos el viernes")

  println(email1 == email2) // true
}
```

Aunque `email1` y `email2` son objetos distintos en memoria, sus datos son iguales. Por eso el resultado es `true`.

En una clase normal, esa comparación no funciona así automáticamente.

## Crear copias con `copy()`

`copy()` permite crear una copia del objeto cambiando solo algunas propiedades.

```kotlin
data class Email(
  val asunto: String,
  val remitente: String,
  val mensaje: String
)

fun main() {
  val emailOriginal = Email(
    asunto = "Reunión",
    remitente = "ana@mail.com",
    mensaje = "Nos vemos el viernes"
  )

  val emailActualizado = emailOriginal.copy(
    asunto = "Reunión actualizada"
  )

  println(emailOriginal)
  println(emailActualizado)
}
```

`emailActualizado` conserva el mismo remitente y mensaje, pero cambia el asunto.

Esto es muy útil cuando querés mantener los datos originales y crear una versión modificada.

## Desestructuración

Una dataclase permite extraer sus valores en variables separadas.

```kotlin
data class Contacto(
  val nombre: String,
  val email: String
)

fun main() {
  val contacto = Contacto("María", "maria@mail.com")

  val (nombre, email) = contacto

  println(nombre)
  println(email)
}
```

Esto funciona porque Kotlin genera funciones internas como `component1()` y `component2()`.

## `hashCode()`

Las dataclases también generan `hashCode()` automáticamente.

```kotlin
val contacto = Contacto("María", "maria@mail.com")

println(contacto.hashCode())
```

El hash se calcula a partir de las propiedades del objeto.

Ojo: un hash ayuda a comparar y organizar objetos en estructuras como mapas o sets, pero no debe pensarse como un identificador único perfecto. Dos objetos diferentes podrían llegar a tener el mismo hash.

## ¿Cuándo usar una dataclase?

Usá una `data class` cuando la clase representa principalmente datos.

Ejemplos:

- un email;
- un contacto;
- un usuario;
- un producto;
- una respuesta de una API.

```kotlin
data class Usuario(
  val id: Int,
  val nombre: String,
  val email: String
)
```

## ¿Cuándo usar una clase normal?

Usá una clase normal cuando el comportamiento pesa más que los datos.

Por ejemplo, cuando:

- hay mucha lógica interna;
- necesitás validar muchos estados;
- el objeto tiene métodos complejos;
- la clase coordina varias operaciones.

## Idea clave

Una `data class` es ideal para transportar y comparar datos.

Una clase normal es mejor cuando necesitás modelar comportamiento más complejo.

No uses `data class` porque sí. Usala cuando el objetivo principal de la clase sea representar información.

-----

## Resumen en 5 líneas

1. Se declara con `data class Nombre(val a: Tipo, ...)`.
2. `==` compara el contenido de dos data classes, no si son el mismo objeto.
3. `copy()` crea una copia cambiando solo las propiedades que indiques.
4. La desestructuración (`val (asunto, remitente) = email`) saca varias propiedades en una línea.
5. Úsala cuando la clase representa información; si pesa más el comportamiento, usa una clase normal.

-----

## Siguiente lección

Sigue con: [Enums](04-enums.md).
