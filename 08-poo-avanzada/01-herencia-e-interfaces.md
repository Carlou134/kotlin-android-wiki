# Herencia e interfaces

## En una frase

La herencia permite que una clase reutilice y especialice a otra; una interfaz es un **contrato** ("cualquiera que me implemente sabe hacer esto") que muchas clases distintas pueden cumplir, y es la herramienta más usada para desacoplar código.

-----

## Antes de empezar

Conviene que ya sepas:

* Crear clases, propiedades y funciones miembro: [Clases](../06-poo/01-clases.md).

Palabras nuevas (también están en el [Glosario](Glosario.md)):

* **Herencia:** Mecanismo por el que una clase (hija) recibe las propiedades y funciones de otra (padre).
* **`open`:** Marca una clase o función para que se pueda heredar o sobrescribir. En Kotlin, por defecto, todo es `final`.
* **`abstract`:** Clase que no se puede instanciar, o función sin cuerpo que las hijas deben implementar.
* **`override`:** Palabra clave para reemplazar una función o propiedad heredada.
* **Interfaz:** Contrato con funciones (y propiedades) que una clase se compromete a implementar.
* **Polimorfismo:** Usar objetos de clases distintas a través de un mismo tipo, y que cada uno responda a su manera.

-----

## Herencia: clases que se especializan

En Kotlin, las clases son **cerradas por defecto**: no se pueden heredar. Para permitirlo, se marcan con `open`:

```kotlin
open class Empleado(val nombre: String) {
  open fun sueldo(): Double = 1000.0
  fun presentarse() = println("Hola, soy $nombre")
}

class Gerente(nombre: String) : Empleado(nombre) {
  override fun sueldo(): Double = 2500.0
}
```

* `: Empleado(nombre)` dice que `Gerente` hereda de `Empleado` y le pasa el nombre a su constructor.
* `sueldo()` es `open`, así que `Gerente` la puede reemplazar con `override`.
* `presentarse()` no es `open`: las hijas la heredan tal cual y no la pueden cambiar.

```kotlin
val g = Gerente("Ana")
g.presentarse()     // Hola, soy Ana   (heredada)
println(g.sueldo()) // 2500.0          (sobrescrita)
```

¿Por qué Kotlin cierra todo por defecto? Porque heredar de una clase que no fue pensada para eso es frágil: si el padre cambia por dentro, las hijas se rompen. Con `open`, el autor de la clase decide **explícitamente** qué se puede especializar.

## Clases abstractas

Una clase `abstract` es un "padre incompleto": define parte del comportamiento y obliga a las hijas a completar el resto.

```kotlin
abstract class Figura {
  abstract fun area(): Double                   // sin cuerpo: las hijas DEBEN implementarla
  fun describir() = "Figura de área ${area()}"  // usa lo que las hijas definan
}

class Circulo(val radio: Double) : Figura() {
  override fun area() = Math.PI * radio * radio
}

// val f = Figura()   ❌ no compila: no se puede instanciar una clase abstracta
```

## Interfaces: contratos

Una interfaz no dice **qué es** un objeto, sino **qué sabe hacer**:

```kotlin
interface Notificador {
  fun enviar(destinatario: String, mensaje: String)
}

class EmailNotificador : Notificador {
  override fun enviar(destinatario: String, mensaje: String) {
    println("Email a $destinatario: $mensaje")
  }
}

class SmsNotificador : Notificador {
  override fun enviar(destinatario: String, mensaje: String) {
    println("SMS a $destinatario: $mensaje")
  }
}
```

`EmailNotificador` y `SmsNotificador` no tienen nada en común por dentro, pero los dos **cumplen el contrato**.

## Polimorfismo: un tipo, muchos comportamientos

Ahora puedes escribir código que trabaja con **cualquier** `Notificador`, sin saber cuál es:

```kotlin
fun avisarATodos(notificadores: List<Notificador>, mensaje: String) {
  for (n in notificadores) {
    n.enviar("ana@mail.com", mensaje)  // cada uno responde a su manera
  }
}

avisarATodos(listOf(EmailNotificador(), SmsNotificador()), "Tu pedido salió")
```

Mañana agregas `PushNotificador` y `avisarATodos` funciona sin tocar una línea. Esa es la idea central de toda la arquitectura que viene después: el código importante depende del **contrato**, no de las implementaciones.

## Más cosas que puede tener una interfaz

**Funciones con implementación por defecto:**

```kotlin
interface Notificador {
  fun enviar(destinatario: String, mensaje: String)

  fun enviarUrgente(destinatario: String, mensaje: String) {
    enviar(destinatario, "URGENTE: $mensaje")  // las clases la heredan gratis
  }
}
```

**Propiedades** (sin valor guardado; la clase decide cómo darlas):

```kotlin
interface Identificable {
  val id: String
}

data class Cliente(override val id: String, val nombre: String) : Identificable
```

**Varias interfaces a la vez:** una clase hereda de **una sola** clase, pero puede implementar **muchas** interfaces.

```kotlin
class Pedido(override val id: String) : Identificable, Comparable<Pedido> {
  override fun compareTo(other: Pedido) = id.compareTo(other.id)
}
```

## `fun interface`: interfaces de una sola función

Si la interfaz tiene **una sola** función abstracta, puedes marcarla `fun interface` y crear implementaciones con una lambda:

```kotlin
fun interface Validador {
  fun esValido(texto: String): Boolean
}

val noVacio = Validador { it.isNotBlank() }
val esEmail = Validador { it.contains("@") }

println(esEmail.esValido("ana@mail.com")) // true
```

## ¿Clase abstracta o interfaz?

| | Clase abstracta | Interfaz |
| --- | --- | --- |
| ¿Puede guardar estado (propiedades con valor)? | Sí | No |
| ¿Tiene constructor? | Sí | No |
| ¿Cuántas puede tener una clase? | Una | Muchas |
| Expresa | "Es un tipo de..." | "Sabe hacer..." |

Regla práctica: **empieza por una interfaz**. Usa una clase abstracta solo si de verdad necesitas compartir estado o un constructor. En código moderno de Kotlin, las interfaces son mucho más comunes.

## Dónde lo vas a ver

En un backend con arquitectura hexagonal, **todos los puertos son interfaces**: `PedidoRepository`, `CrearPedido`, `Notificador`. Lo verás en [SOLID e inversión de dependencias](../12-arquitectura/01-solid-y-dip.md).

-----

## Resumen en 5 líneas

1. En Kotlin las clases son `final` por defecto; `open` permite heredar y `override` reemplaza lo heredado.
2. Una clase `abstract` no se instancia y obliga a las hijas a implementar sus funciones abstractas.
3. Una interfaz es un contrato: dice qué sabe hacer un objeto, no qué es; puede tener funciones por defecto y propiedades.
4. El polimorfismo permite trabajar con cualquier implementación a través del tipo de la interfaz.
5. Una clase hereda de una sola clase pero implementa muchas interfaces; prefiere interfaces, y `fun interface` para contratos de una función.

-----

## Siguiente lección

[2. Clases e interfaces selladas](02-sealed.md): jerarquías cerradas que el compilador conoce completas.
