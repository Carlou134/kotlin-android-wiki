# Enums en Kotlin: Constantes organizadas para casos específicos

## En una frase

Un enum es una clase con un conjunto fijo de opciones, así el compilador no te deja usar un valor que no existe.

-----

## Antes de empezar

Conviene que ya sepas:

* Un enum es una clase especial: [Clases](01-clases.md).
* La expresión `when`: [Condicionales](../04-control-de-flujo/01-condicionales.md).

Palabras nuevas (también están en el [Glosario](Glosario.md)):

* **`entries`:** Propiedad de un enum que devuelve todas sus opciones, en orden de declaración.
* **Enum:** Clase con un conjunto fijo de valores posibles, declarada con `enum class`.

-----

Un **enum** es una clase especial que representa un grupo fijo de opciones.

Sirve cuando un valor solo puede ser una de varias posibilidades conocidas.

Por ejemplo, una carpeta de email puede ser:

- bandeja de entrada;
- enviados;
- borradores;
- archivados;
- spam.

En vez de usar strings sueltos por todo el código, usamos un enum.

## Declarar un enum

En Kotlin se declara con `enum class`.

```kotlin
enum class EmailFolder {
  INBOX,
  SENT,
  DRAFT,
  ARCHIVE,
  SPAM
}
```

Cada nombre dentro del enum es una constante.

```kotlin
val carpetaActual = EmailFolder.INBOX
```

Esto es más seguro que escribir:

```kotlin
val carpetaActual = "inbox"
```

¿Por qué? Porque con strings podés equivocarte escribiendo `"inboox"` y Kotlin no te protege. Con enums, las opciones están limitadas.

## Usar enums con `when`

Los enums funcionan muy bien con `when`.

```kotlin
fun mostrarCarpeta(carpetaActual: EmailFolder) {
  when (carpetaActual) {
    EmailFolder.INBOX -> println("Bienvenido a bandeja de entrada")
    EmailFolder.SENT -> println("Bienvenido a enviados")
    EmailFolder.DRAFT -> println("Bienvenido a borradores")
    EmailFolder.ARCHIVE -> println("Bienvenido a archivados")
    EmailFolder.SPAM -> println("Bienvenido a spam")
  }
}
```

La ventaja es enorme: Kotlin sabe cuáles son todas las opciones posibles.

Si te falta una rama en el `when`, el compilador puede avisarte cuando el `when` se usa como expresión. ESO es seguridad real, no memoria ni “me acuerdo después”.

## Enums con propiedades

Un enum también puede tener datos propios.

Por ejemplo, una prioridad de email puede tener nivel y color.

```kotlin
enum class EmailPriority(
  val level: Int,
  val color: String
) {
  LOW(1, "gris"),
  NORMAL(2, "azul"),
  HIGH(3, "naranja"),
  URGENT(4, "rojo")
}
```

Ahora cada prioridad tiene información asociada:

```kotlin
val prioridad = EmailPriority.URGENT

println(prioridad.level) // 4
println(prioridad.color) // rojo
```

Esto sirve cuando querés que cada opción tenga datos extra sin crear varias variables separadas.

## Recorrer todos los valores de un enum

Kotlin permite recorrer todas las constantes con `entries`.

```kotlin
EmailPriority.entries.forEachIndexed { index, priority ->
  println("$index: ${priority.name}, nivel ${priority.level}, color ${priority.color}")
}
```

Salida aproximada:

```text
0: LOW, nivel 1, color gris
1: NORMAL, nivel 2, color azul
2: HIGH, nivel 3, color naranja
3: URGENT, nivel 4, color rojo
```

Esto es útil para mostrar opciones al usuario.

## Convertir una entrada numérica a enum

Si el usuario elige una opción por número, primero hay que validar que ese número exista.

```kotlin
fun prioridadAPriority(numero: Int): EmailPriority? {
  return if (numero in 0 until EmailPriority.entries.size) {
    EmailPriority.entries[numero]
  } else {
    null
  }
}
```

El tipo de retorno es `EmailPriority?` porque puede devolver:

- una prioridad válida;
- `null` si el número está fuera del rango.

Ejemplo:

```kotlin
val prioridad = prioridadAPriority(2)

println(prioridad) // HIGH
```

Si llamás `prioridadAPriority(10)`, devuelve `null` porque no existe esa posición.

## Ejercicio: estado de un email

Creemos un enum para representar el estado de un correo.

```kotlin
enum class EstadoEmail(
  val icono: String
) {
  NUEVO("📩"),
  LEIDO("✅"),
  RESPONDIDO("↩️")
}
```

Ahora hacemos una función para convertir un número en un estado:

```kotlin
fun numeroAEstado(numero: Int): EstadoEmail? {
  return if (numero in 0 until EstadoEmail.entries.size) {
    EstadoEmail.entries[numero]
  } else {
    null
  }
}
```

Mostramos las opciones disponibles:

```kotlin
fun mostrarEstados() {
  EstadoEmail.entries.forEachIndexed { index, estado ->
    println("$index: ${estado.name} ${estado.icono}")
  }
}
```

Ejemplo completo:

```kotlin
fun main() {
  mostrarEstados()

  val estado = numeroAEstado(1)

  if (estado != null) {
    println("Estado seleccionado: ${estado.name} ${estado.icono}")
  } else {
    println("Estado inválido")
  }
}
```

## ¿Cuándo usar enums?

Usá enums cuando tenés un conjunto fijo de opciones.

Ejemplos:

- estados: `NUEVO`, `LEIDO`, `RESPONDIDO`;
- prioridades: `LOW`, `NORMAL`, `HIGH`, `URGENT`;
- carpetas: `INBOX`, `SENT`, `DRAFT`, `SPAM`;
- roles: `ADMIN`, `USER`, `GUEST`.

## Idea clave

Un enum limita las opciones posibles y hace que el código sea más claro y seguro.

Si un dato solo puede tener ciertos valores concretos, NO lo dejes como string suelto. Modelalo con un enum.

-----

## Resumen en 5 líneas

1. Se declara con `enum class Estado { NUEVO, LEIDO, RESPONDIDO }`.
2. Evita strings sueltos: con un enum, escribir mal una opción es un error de compilación y no un bug en ejecución.
3. Con `when` manejas cada opción; si el `when` devuelve un valor y cubre todas las opciones, no necesita `else`.
4. Un enum puede tener propiedades definidas en su constructor, como `enum class Prioridad(val nivel: Int)`.
5. `entries` devuelve todas las opciones para recorrerlas o convertir un número en una opción.

-----

## Siguiente lección

Terminaste esta carpeta. Sigue con la siguiente: [Null safety y errores](../07-null-safety-y-errores/README.md).
