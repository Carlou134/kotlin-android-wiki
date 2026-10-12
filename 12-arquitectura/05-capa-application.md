# La capa application: use cases, commands y outcomes

## En una frase

La capa application orquesta el negocio: recibe una intención (**command**), usa el dominio y los puertos de salida, y devuelve un resultado cerrado (**outcome**) que obliga al que llama a manejar cada caso posible.

-----

## Antes de empezar

Conviene que ya sepas:

* Las capas y los puertos: [Arquitectura hexagonal](02-arquitectura-hexagonal.md).
* Excepciones y cuándo usarlas: [Excepciones](../07-null-safety-y-errores/02-excepciones.md).
* `sealed interface` y `when` exhaustivo: [Clases e interfaces selladas](../08-poo-avanzada/02-sealed.md).

Palabras nuevas (también están en el [Glosario](Glosario.md)):

* **Use case:** Una operación de negocio completa, como "crear pedido" o "cancelar pedido".
* **Command:** Data class con los datos de entrada de un use case.
* **Outcome:** Tipo `sealed` con todos los resultados posibles de un use case.
* **Error de negocio:** Un resultado esperado que no es el feliz ("sin stock"), a diferencia de un fallo técnico ("se cayó la base").

-----

## El problema: errores que nadie ve venir

Mirá esta firma:

```kotlin
fun cancelarPedido(id: String): Pedido
```

¿Qué pasa si el pedido no existe? ¿Y si ya fue enviado? La firma **no lo dice**. Probablemente lance excepciones, y quien llama se entera en producción.

## Commands: la entrada con nombre

En vez de una lista suelta de parámetros, un command agrupa la intención:

```kotlin
data class CancelarPedidoCommand(
  val pedidoId: PedidoId,
  val solicitadoPor: ClienteId,
  val motivo: String,
)
```

Ventajas: el nombre dice qué se quiere hacer, puedes agregar campos sin romper todas las llamadas y los tests se leen mejor.

## Outcomes: la salida honesta

Un `sealed interface` lista **todos** los resultados posibles:

```kotlin
sealed interface CancelarPedidoOutcome {
  data class Cancelado(val pedido: Pedido) : CancelarPedidoOutcome
  data object NoEncontrado : CancelarPedidoOutcome
  data object NoEsTuyo : CancelarPedidoOutcome
  data class YaEnviado(val enviadoEl: Instant) : CancelarPedidoOutcome
}
```

Ahora la firma no miente:

```kotlin
interface CancelarPedido {
  fun ejecutar(command: CancelarPedidoCommand): CancelarPedidoOutcome
}
```

## El use case

```kotlin
class CancelarPedidoService(
  private val pedidos: PedidoRepository,
) : CancelarPedido {

  override fun ejecutar(command: CancelarPedidoCommand): CancelarPedidoOutcome {
    val pedido = pedidos.buscar(command.pedidoId)
      ?: return CancelarPedidoOutcome.NoEncontrado

    if (pedido.clienteId != command.solicitadoPor) {
      return CancelarPedidoOutcome.NoEsTuyo
    }

    val enviadoEl = pedido.enviadoEl
    if (enviadoEl != null) {
      return CancelarPedidoOutcome.YaEnviado(enviadoEl)
    }

    val cancelado = pedido.cancelar(command.motivo)
    pedidos.guardar(cancelado)
    return CancelarPedidoOutcome.Cancelado(cancelado)
  }
}
```

Fíjate en el patrón de **guard clauses**: cada condición inválida sale temprano con su outcome. Y con `?: return`, el compilador sabe que después de esa línea `pedido` no es `null`, sin usar `!!`.

## Quien llama está obligado a manejar todo

En la ruta HTTP, `when` sobre un tipo `sealed` es **exhaustivo**: si mañana agregas un outcome nuevo y te olvidas de manejarlo, no compila.

```kotlin
when (val outcome = cancelarPedido.ejecutar(command)) {
  is CancelarPedidoOutcome.Cancelado -> call.respond(HttpStatusCode.OK, outcome.pedido.toResponse())
  CancelarPedidoOutcome.NoEncontrado -> call.respond(HttpStatusCode.NotFound)
  CancelarPedidoOutcome.NoEsTuyo -> call.respond(HttpStatusCode.Forbidden)
  is CancelarPedidoOutcome.YaEnviado -> call.respond(HttpStatusCode.Conflict)
}
```

Fíjate también en el reparto de responsabilidades: el use case habla en **lenguaje de negocio** (`YaEnviado`); traducir eso a un código HTTP (`409 Conflict`) es trabajo del adapter.

## ¿Outcomes o excepciones?

| Situación | Usar |
| --- | --- |
| Resultado esperado del negocio: no existe, sin permiso, estado inválido | **Outcome** |
| Fallo técnico inesperado: se cayó la base, timeout de red | **Excepción** |
| Bug del programador: un invariante roto | **Excepción** (`require`, `check`, `error`) |

Los errores de negocio son parte del contrato; las excepciones son para lo que no debería pasar.

## Convenciones de nombres

Un esquema de nombres consistente hace que el código se navegue solo:

| Pieza | Nombre | Ejemplo |
| --- | --- | --- |
| Puerto de entrada | Verbo + sustantivo | `CancelarPedido` |
| Implementación | Lo mismo + `Service` | `CancelarPedidoService` |
| Entrada | Lo mismo + `Command` | `CancelarPedidoCommand` |
| Salida | Lo mismo + `Outcome` | `CancelarPedidoOutcome` |
| Puerto de salida | Sustantivo + rol | `PedidoRepository`, `NotificadorPort` |

-----

## Resumen en 5 líneas

1. La capa application orquesta: recibe un command, usa el dominio y los puertos, devuelve un outcome.
2. Un command es una data class con la intención y los datos de entrada.
3. Un outcome es un `sealed interface` con todos los resultados posibles, así la firma no esconde errores.
4. El `when` exhaustivo obliga a quien llama a manejar cada caso; el adapter traduce outcomes a HTTP.
5. Outcomes para errores de negocio esperados; excepciones para fallos técnicos y bugs.

-----

## Siguiente lección

[6. Tests de arquitectura](06-tests-de-arquitectura.md): cómo hacer que estas reglas no dependan de la memoria de nadie.
