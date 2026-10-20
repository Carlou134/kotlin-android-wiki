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
* **Repository:** Interfaz que define cómo guardar, buscar y borrar datos, sin decir dónde se guardan.

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

## Un caso real: el patrón repository

Un **repository** es una interfaz que define cómo guardar, buscar y borrar datos, **sin decir dónde** se guardan (una lista en memoria, una base de datos, una API). Es el uso más común de las interfaces en un backend o en una app.

### El contrato

```kotlin
import java.util.UUID

data class Email(val id: UUID, val asunto: String, val cuerpo: String)

interface EmailRepository {
  val cantidad: Int
  fun guardar(email: Email)
  fun buscarPorId(id: UUID): Email?
  fun listarTodos(): List<Email>
  fun borrarPorId(id: UUID)
}
```

* `UUID` es una clase de Java: desde Kotlin puedes usar cualquier clase de la librería estándar de Java. (Kotlin 2.0.20 agregó su propia `kotlin.uuid.Uuid`, todavía experimental.)
* `buscarPorId` devuelve `Email?`: el contrato **dice explícitamente** que puede no encontrar nada. Quien lo use está obligado a manejar el `null`.
* `cantidad` es una **propiedad** del contrato: quien la usa no sabe ni le importa si detrás hay una lista o un `SELECT COUNT(*)`.
* No hay ni una línea de lógica. Solo firmas.

### Una implementación

```kotlin
class InMemoryEmailRepository : EmailRepository {
  private val emails = mutableListOf<Email>()

  override val cantidad: Int
    get() = emails.size

  override fun guardar(email: Email) {
    emails.add(email)
  }

  override fun buscarPorId(id: UUID): Email? = emails.find { it.id == id }

  override fun listarTodos(): List<Email> = emails.toList()

  override fun borrarPorId(id: UUID) {
    emails.removeAll { it.id == id }
  }
}
```

Varias cosas para notar:

* **`: EmailRepository`** usa la misma sintaxis que la herencia, pero sin paréntesis: una interfaz no tiene constructor.
* **`override` es obligatorio** en cada miembro que viene del contrato. Sin él, no compila.
* **Si te falta implementar algo**, el compilador lo marca: *Class 'InMemoryEmailRepository' is not abstract and does not implement abstract member...*. En IntelliJ, `Alt + Enter` (en Mac, `Option + Enter`) → *Implement members* genera los que faltan.
* **La lista es `private`**: nadie de afuera puede tocarla sin pasar por el contrato. Es el [encapsulamiento](../06-poo/05-visibilidad-y-encapsulamiento.md) en acción.
* **`listarTodos()` devuelve `emails.toList()`**, una copia. Si devolviera `emails` directamente, quien llama podría castearla a `MutableList` y modificarla por fuera.
* **`removeAll { }` en vez de `removeIf { }`**: los dos funcionan en la JVM, pero `removeIf` es de Java y `removeAll` es de Kotlin, y también funciona fuera de la JVM.

### Nombra la implementación por lo que es

Evita `EmailRepositoryImpl` o `EmailRepositoryImplementation`: no dicen nada. `InMemoryEmailRepository` dice **cómo** funciona. Mañana agregas `PostgresEmailRepository` o `RoomEmailRepository` y los nombres no chocan.

### Usarlo a través del contrato

```kotlin
fun main() {
  val repositorio: EmailRepository = InMemoryEmailRepository()

  println(repositorio.cantidad)   // 0
  repositorio.guardar(Email(UUID.randomUUID(), "Reunión", "Mañana a las 10"))
  println(repositorio.cantidad)   // 1

  val primero = repositorio.listarTodos().first()
  repositorio.borrarPorId(primero.id)   // el id es aleatorio: hay que sacarlo del repositorio
  println(repositorio.cantidad)   // 0
}
```

La variable se declara con el tipo **de la interfaz** (`: EmailRepository`). Para cambiar a una base de datos, solo cambias lo que va después del `=`; el resto del código no se entera.

### El tipo declarado decide qué puedes ver

Si `InMemoryEmailRepository` tuviera algo extra, fuera del contrato:

```kotlin
class InMemoryEmailRepository : EmailRepository {
  val nombre = "memoria"
  // ...
}

val repo: EmailRepository = InMemoryEmailRepository()
// repo.nombre   ❌ no compila: EmailRepository no tiene "nombre"

val repoConcreto = InMemoryEmailRepository()
repoConcreto.nombre   // ✅ acá el tipo es la clase concreta
```

El objeto es el mismo, pero el compilador solo te deja usar lo que dice el **tipo declarado**. Esa limitación es justamente lo que te protege: si tu código solo usa el contrato, puedes cambiar la implementación sin romper nada.

En una app o un backend real, la implementación no se crea a mano en cada lugar: se arma en un solo punto y se "inyecta". Eso se ve en [Composition root](../12-arquitectura/04-composition-root.md) y con [Koin](../14-ktor-y-koin/04-koin.md).

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

## Ejercicio: un notificador intercambiable

1. Crea una interfaz `EmailNotifier` con una función `notificar(email: Email)`.
2. Crea `ConsoleNotifier`, que la implemente imprimiendo `"Nuevo email: ${email.asunto}"`.
3. En `main`, declara `val notifier: EmailNotifier = ConsoleNotifier()` y llama a `notifier.notificar(...)`.

<details>
<summary>Ver solución</summary>

```kotlin
interface EmailNotifier {
  fun notificar(email: Email)
}

class ConsoleNotifier : EmailNotifier {
  override fun notificar(email: Email) {
    println("Nuevo email: ${email.asunto}")
  }
}

fun main() {
  val notifier: EmailNotifier = ConsoleNotifier()
  notifier.notificar(Email(UUID.randomUUID(), "Oferta", "Tenemos una propuesta"))
}
```

</details>

La gracia: en Android podrías crear un `AndroidNotifier` que muestre la notificación en el sistema, y el código que llama a `notificar` no cambia. Como tiene una sola función, `EmailNotifier` también podría ser una [`fun interface`](#fun-interface-interfaces-de-una-sola-función).

## Dónde lo vas a ver

En un backend con arquitectura hexagonal, **todos los puertos son interfaces**: `PedidoRepository`, `CrearPedido`, `Notificador`. Lo verás en [SOLID e inversión de dependencias](../12-arquitectura/01-solid-y-dip.md).

-----

## Resumen en 5 líneas

1. En Kotlin las clases son `final` por defecto; `open` permite heredar y `override` reemplaza lo heredado.
2. Una clase `abstract` no se instancia y obliga a las hijas a implementar sus funciones abstractas.
3. Una interfaz es un contrato: dice qué sabe hacer un objeto, no qué es; puede tener funciones por defecto y propiedades.
4. El polimorfismo permite trabajar con cualquier implementación a través del tipo de la interfaz; el patrón repository (`EmailRepository` + `InMemoryEmailRepository`) es el caso típico, y el tipo declarado limita lo que puedes usar.
5. Una clase hereda de una sola clase pero implementa muchas interfaces; prefiere interfaces, y `fun interface` para contratos de una función.

-----

## Siguiente lección

[2. Clases e interfaces selladas](02-sealed.md): jerarquías cerradas que el compilador conoce completas.
