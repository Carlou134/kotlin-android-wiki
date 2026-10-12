# `value class`, `object` y `companion object`

## En una frase

Una `value class` envuelve un solo valor en un tipo propio (como `PedidoId` en vez de `String`) para que el compilador no te deje mezclarlos, casi sin costo; un `object` es una clase con una **única** instancia, y un `companion object` agrupa lo que pertenece a la clase y no a cada instancia.

-----

## Antes de empezar

Conviene que ya sepas:

* Data classes y el bloque `init`: [Data classes](../06-poo/03-data-classes.md) y [Clases](../06-poo/01-clases.md).
* `require` y excepciones: [Excepciones](../07-null-safety-y-errores/02-excepciones.md).

Palabras nuevas (también están en el [Glosario](Glosario.md)):

* **`value class`:** Clase que envuelve un único valor y, en la mayoría de los casos, no crea un objeto extra en tiempo de ejecución.
* **Boxing:** Cuando Kotlin sí tiene que crear el objeto envoltorio (por ejemplo, al usarlo como nulable o en genéricos).
* **Singleton:** Un tipo del que existe una sola instancia en todo el programa.
* **`object`:** Declaración de Kotlin que crea un singleton.
* **`companion object`:** Objeto dentro de una clase cuyos miembros se usan con el nombre de la clase.
* **Función de fábrica:** Función que crea instancias, en vez de llamar al constructor directamente.

-----

## El problema: todo es `String`

```kotlin
fun transferir(origenId: String, destinoId: String, monto: Double) { /* ... */ }

transferir(destinoId, origenId, 500.0)  // ❌ invertidos: compila perfecto
```

Los dos parámetros son `String`, así que el compilador no puede ayudarte. Lo mismo pasa con un `clienteId` pasado donde se esperaba un `pedidoId`. Son bugs silenciosos que aparecen en producción.

## La solución: `value class`

```kotlin
@JvmInline
value class CuentaId(val value: String)

fun transferir(origen: CuentaId, destino: CuentaId, monto: Double) { /* ... */ }

val a = CuentaId("c-1")
val pedido = PedidoId("p-9")
transferir(a, pedido, 500.0)  // ❌ no compila: PedidoId no es CuentaId
```

* Debe tener **exactamente una** propiedad, `val`, en el constructor.
* `@JvmInline` es obligatorio cuando se compila para la JVM.

## ¿Por qué "casi sin costo"?

Una `data class CuentaId(val value: String)` también resolvería el problema, pero crea un objeto extra por cada ID. Una `value class`, en la mayoría de los casos, se **compila como el valor que envuelve**: en tiempo de ejecución es un `String` normal, y solo el compilador ve el tipo `CuentaId`.

Hay casos en que sí se crea el envoltorio (**boxing**): cuando la usas como tipo nulable (`CuentaId?`), dentro de genéricos (`List<CuentaId>`) o a través de una interfaz. Funciona igual; solo pierde la ventaja de rendimiento.

## Validar al crear

Como cualquier clase, una `value class` puede tener un `init`. Así, un valor inválido **no puede existir**:

```kotlin
@JvmInline
value class Email(val value: String) {
  init {
    require(value.contains("@")) { "Email inválido: $value" }
  }
}

val ok = Email("ana@mail.com")
val mal = Email("ana")  // lanza IllegalArgumentException
```

Cualquier función que recibe un `Email` puede confiar en que es válido. No tiene que volver a validarlo.

## `object`: una sola instancia

A veces solo necesitas **una** instancia de algo:

```kotlin
object ContadorDeVisitas {
  private var total = 0
  fun registrar() { total++ }
  fun total() = total
}

ContadorDeVisitas.registrar()
println(ContadorDeVisitas.total()) // 1
```

* No tiene constructor: no se puede instanciar con `ContadorDeVisitas()`.
* Se crea la primera vez que se usa.
* Se usa directamente por su nombre.

Cuidado con los `object` que guardan estado que cambia: son variables globales disfrazadas, y vuelven el código difícil de testear. Úsalos para cosas sin estado o constantes, o para describir algo fijo.

Ya viste un `object` sin saberlo: en Exposed, cada tabla es un `object`, porque describe algo único.

```kotlin
object PedidosTable : Table("pedidos") { /* ... */ }
```

## `data object`

En la lección anterior usaste `data object` para los casos sin datos de un tipo `sealed`. Es un `object` con un `toString()` legible (`NoEncontrado` en vez de `NoEncontrado@1b6d3586`) y un `equals` correcto.

## `companion object`: lo que es de la clase

Kotlin no tiene `static`. Lo que pertenece a la **clase** (y no a cada instancia) va en un `companion object`:

```kotlin
class Pedido private constructor(val id: PedidoId, val total: Double) {

  companion object {
    const val TOTAL_MAXIMO = 1_000_000.0

    fun crear(total: Double): Pedido {
      require(total in 0.01..TOTAL_MAXIMO) { "Total fuera de rango: $total" }
      return Pedido(PedidoId(UUID.randomUUID().toString()), total)
    }
  }
}

val p = Pedido.crear(150.0)        // se llama con el nombre de la clase
println(Pedido.TOTAL_MAXIMO)
// Pedido(...)                     ❌ el constructor es privado
```

Dos usos típicos:

* **Funciones de fábrica:** con el constructor `private`, la única forma de crear un `Pedido` es `Pedido.crear(...)`, que valida y genera el ID. Así se protegen las reglas de creación.
* **Constantes:** `const val` para valores fijos conocidos al compilar.

Otro uso común es convertir desde otro formato, con un nombre que explica el origen:

```kotlin
data class DbConfig(val url: String, val usuario: String) {
  companion object {
    fun desdeEntorno() = DbConfig(
      url = System.getenv("DB_URL") ?: "jdbc:postgresql://localhost:5432/tienda",
      usuario = System.getenv("DB_USER") ?: "tienda",
    )
  }
}
```

## Dónde lo vas a ver

* IDs tipados (`PedidoId`, `ClienteId`, `EmpresaId`) en todo el dominio: [Arquitectura hexagonal](../12-arquitectura/02-arquitectura-hexagonal.md) y [DDD y monolito modular](../12-arquitectura/03-ddd-y-monolito-modular.md).
* Tablas de Exposed como `object`: [Exposed DSL](../15-persistencia/04-exposed-dsl.md).
* `data object` en outcomes: [La capa application](../12-arquitectura/05-capa-application.md).

-----

## Resumen en 5 líneas

1. Una `value class` envuelve un solo valor en un tipo propio, y el compilador impide mezclar, por ejemplo, un `PedidoId` con un `CuentaId`.
2. En la mayoría de los casos no cuesta nada en tiempo de ejecución; hay boxing en nulables, genéricos e interfaces.
3. Con un `init` que valida, un valor inválido no puede existir.
4. Un `object` es un singleton; evita que guarde estado cambiante, y usa `data object` para casos sin datos.
5. El `companion object` reemplaza a `static`: funciones de fábrica (con constructor privado) y constantes.

-----

## Siguiente lección

Terminaste la POO avanzada. Lo próximo es la [programación funcional](../09-funcional/README.md). Vuelve al índice: [README principal](../README.md).
