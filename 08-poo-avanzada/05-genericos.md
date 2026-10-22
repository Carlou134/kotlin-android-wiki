# Genéricos (`<T>`)

## En una frase

Un genérico es una clase, interfaz o función que recibe un **tipo como parámetro** (`<T>`), así escribes la lógica una sola vez y la usas con `Email`, `Contacto` o cualquier otro tipo, sin perder la seguridad del compilador.

-----

## Antes de empezar

Conviene que ya sepas:

* Interfaces y el patrón repository: [Herencia e interfaces](01-herencia-e-interfaces.md).
* Maps: [Maps](../03-colecciones/03-maps.md).
* Tipos nulables: [Nulabilidad](../07-null-safety-y-errores/01-nulabilidad.md).

Palabras nuevas (también están en el [Glosario](Glosario.md)):

* **Genérico:** Clase, interfaz o función que trabaja con un tipo que se decide al usarla.
* **Parámetro de tipo:** El "hueco" para el tipo, escrito entre `< >`. Por convención, `T` (de *type*).
* **Argumento de tipo:** El tipo concreto que llena el hueco al usarlo, como `Email` en `Repository<Email>`.
* **Restricción (upper bound):** Condición sobre el parámetro de tipo, como `<T : Identificable>`: solo se aceptan tipos que cumplan ese contrato.

-----

## Ya los usaste sin saberlo

```kotlin
val nombres: List<String> = listOf("Ana", "Luis")
val stock: Map<String, Int> = mapOf("teclado" to 5)
```

`List` no es una lista "de strings": es una lista **de T**, y `String` es el tipo que elegiste. Los autores de `List` escribieron la lógica una vez; tú decides con qué la usas.

## El problema: el mismo repositorio, copiado

En [Herencia e interfaces](01-herencia-e-interfaces.md#un-caso-real-el-patrón-repository) armaste un `EmailRepository`. Mañana necesitas guardar contactos:

```kotlin
interface EmailRepository {
  fun guardar(email: Email)
  fun buscarPorId(id: String): Email?
  fun listarTodos(): List<Email>
}

interface ContactoRepository {
  fun guardar(contacto: Contacto)
  fun buscarPorId(id: String): Contacto?
  fun listarTodos(): List<Contacto>
}
```

Son **idénticos** salvo el tipo. Y sus implementaciones en memoria también lo serían. Podrías usar `Any` para todo, pero entonces `buscarPorId` devolvería `Any?` y tendrías que castear a mano, con el riesgo de equivocarte en tiempo de ejecución.

## Una interfaz genérica

```kotlin
interface Repository<T> {
  fun guardar(id: String, elemento: T)
  fun buscarPorId(id: String): T?
  fun listarTodos(): List<T>
}
```

`T` es un parámetro, igual que `id`, pero en vez de un valor recibe un **tipo**. `Repository<Email>` tiene un `buscarPorId` que devuelve `Email?`; `Repository<Contacto>`, uno que devuelve `Contacto?`.

## Una clase genérica

```kotlin
class InMemoryRepository<T> : Repository<T> {
  private val items = mutableMapOf<String, T>()

  override fun guardar(id: String, elemento: T) {
    items[id] = elemento
  }

  override fun buscarPorId(id: String): T? = items[id]

  override fun listarTodos(): List<T> = items.values.toList()
}
```

* `InMemoryRepository<T> : Repository<T>`: la clase recibe un `T` y se lo pasa a la interfaz.
* Por dentro usa un **map** `id → elemento`, no una lista. Con una lista, `buscarPorId` tendría que recorrer todos los elementos hasta encontrarlo; el map va directo a la clave (usa el `hashCode` del id), así que tarda lo mismo con 10 o con un millón de elementos. Si el orden importa, usa una lista; si buscas por clave, un map.
* `items[id]` ya devuelve `T?`: si la clave no existe, da `null`.
* `listarTodos()` devuelve una copia con `toList()`, por la misma razón que en el repository de la lección 1.

## Usarlo

```kotlin
import java.util.UUID

data class Email(val asunto: String, val cuerpo: String)
data class Contacto(val nombre: String, val correo: String)

fun main() {
  val emails = InMemoryRepository<Email>()
  emails.guardar(UUID.randomUUID().toString(), Email("Reunión", "Mañana a las 10"))
  println(emails.listarTodos())   // [Email(asunto=Reunión, cuerpo=Mañana a las 10)]

  val contactos: Repository<Contacto> = InMemoryRepository()   // T se infiere del tipo declarado
  contactos.guardar("c1", Contacto("Ana", "ana@mail.com"))
  val ana: Contacto? = contactos.buscarPorId("c1")             // sin castear nada

  // emails.guardar("x", Contacto("Luis", "luis@mail.com"))    ❌ no compila: esperaba un Email
}
```

Una sola clase, dos repositorios, y el compilador **sigue** controlando que no mezcles tipos.

## Restricciones: `<T : Algo>`

En `guardar(id, elemento)` hay que pasar el id por separado. ¿Y si el elemento ya trae su id? El problema es que, dentro de `InMemoryRepository<T>`, `T` puede ser **cualquier cosa**: el compilador no sabe si tiene `.id`.

La solución es una **restricción**: aceptar solo tipos que cumplan un contrato.

```kotlin
interface Identificable {
  val id: String
}

class InMemoryRepository<T : Identificable> {
  private val items = mutableMapOf<String, T>()

  fun guardar(elemento: T) {
    items[elemento.id] = elemento   // ✅ el compilador sabe que T tiene "id"
  }

  fun buscarPorId(id: String): T? = items[id]
}

data class Email(override val id: String, val asunto: String) : Identificable

val repo = InMemoryRepository<Email>()   // ✅ Email es Identificable
// InMemoryRepository<String>()          ❌ no compila: String no es Identificable
```

`<T : Identificable>` se lee "cualquier `T`, siempre que sea un `Identificable`". Sin restricción, el límite implícito es `Any?`.

## Funciones genéricas

No solo las clases: una función también puede tener su propio `<T>`, que va **antes** del nombre.

```kotlin
fun <T> primeroODefecto(lista: List<T>, defecto: T): T =
  if (lista.isEmpty()) defecto else lista[0]

val n = primeroODefecto(listOf(3, 7), 0)          // T = Int
val s = primeroODefecto(emptyList<String>(), "-") // T = String
```

Casi nunca escribes el tipo: Kotlin lo infiere de los argumentos. Toda la librería de colecciones (`map`, `filter`, `first`...) está hecha así.

## Un límite: en la JVM, `T` desaparece al ejecutar

Los genéricos se comprueban **al compilar**. Al ejecutar, en la JVM, una `List<Email>` es solo una `List`: el tipo se **borra** (*type erasure*). Por eso esto no compila:

```kotlin
fun <T> esDelTipo(valor: Any): Boolean = valor is T   // ❌ Cannot check for instance of erased type: T
```

La salida es `inline` + `reified`, que se ve en [Funciones `inline`](../09-funcional/03-inline.md#reified-tipos-genéricos-en-tiempo-de-ejecución).

## Dónde lo vas a ver

* `call.receive<T>()` en Ktor y `get<T>()` en Koin: [Funciones `inline`](../09-funcional/03-inline.md) y [Koin](../14-ktor-y-koin/04-koin.md).
* `Result<T>`, `Flow<T>` y, en Android, `StateFlow<UiState>`: el estado de pantalla sellado de la [lección anterior](02-sealed.md) envuelto en un genérico.

-----

## Resumen en 5 líneas

1. Un genérico recibe un tipo como parámetro (`<T>`) para escribir la lógica una vez y usarla con muchos tipos.
2. `List<T>` y `Map<K, V>` ya son genéricos; `Repository<T>` + `InMemoryRepository<T>` evita copiar un repositorio por entidad.
3. A diferencia de usar `Any`, el compilador sigue controlando los tipos: no hay casts ni mezclas posibles.
4. Una restricción `<T : Identificable>` acepta solo tipos que cumplen un contrato y te deja usar sus miembros dentro.
5. En la JVM los tipos genéricos se borran al ejecutar; para consultarlos hace falta `inline` + `reified`.

-----

## Siguiente lección

Terminaste la POO avanzada. Lo próximo es la [programación funcional](../09-funcional/README.md). Vuelve al índice: [README principal](../README.md).
