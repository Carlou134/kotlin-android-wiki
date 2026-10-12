# Arquitectura hexagonal (Ports & Adapters)

## En una frase

La arquitectura hexagonal pone la lógica de negocio en el centro y toda la tecnología (HTTP, base de datos, APIs externas) en el borde, conectadas solo a través de interfaces llamadas **puertos**.

-----

## Antes de empezar

Conviene que ya sepas:

* Qué es la inversión de dependencias: [SOLID e inversión de dependencias](01-solid-y-dip.md).
* Qué son las data classes: [Data classes](../06-poo/03-data-classes.md).

Palabras nuevas (también están en el [Glosario](Glosario.md)):

* **Puerto (port):** Interfaz que define cómo se habla con el núcleo. Hay de entrada y de salida.
* **Adapter:** Implementación concreta que conecta un puerto con una tecnología.
* **Adapter primario (driving):** El que *llama* al núcleo, como una ruta HTTP.
* **Adapter secundario (driven):** El que *es llamado* por el núcleo, como un repositorio de base de datos.
* **DTO:** Objeto que solo transporta datos hacia afuera o desde afuera (JSON, filas de tabla).

-----

## El dibujo

```
            ┌──────────────────────── infrastructure ────────────────────────┐
            │                                                                │
  HTTP ──>  │  Rutas Ktor ──> [CrearPedido] ──> CrearPedidoService           │
            │  (adapter       (puerto de        │                            │
            │   primario)      entrada)         ▼                            │
            │                         [PedidoRepository] <── ExposedPedido-  │ ──> PostgreSQL
            │                         (puerto de salida)     Repository      │
            │                                                (adapter        │
            │                                                 secundario)    │
            └────────────────────────────────────────────────────────────────┘
```

El hexágono no tiene nada de especial: es solo un dibujo con muchos lados para mostrar que el núcleo puede tener **muchas entradas y salidas**, todas iguales en importancia.

## Las tres capas

| Capa | Qué contiene | De qué puede depender |
| --- | --- | --- |
| **domain** | Entidades, value objects y reglas de negocio puras | De nada (solo Kotlin estándar) |
| **application** | Use cases y puertos (de entrada y de salida) | Solo de `domain` |
| **infrastructure** | Adapters: rutas HTTP, repositorios, clientes HTTP | De `application` y `domain` |

La **regla de dependencia**: las flechas siempre apuntan hacia adentro. `domain` no sabe que existe Ktor. `application` no sabe que existe Exposed.

## Recorrido completo con un ejemplo

### domain: el negocio puro

```kotlin
@JvmInline
value class PedidoId(val value: String)

data class Pedido(
  val id: PedidoId,
  val clienteId: String,
  val total: Double,
) {
  init {
    require(total > 0) { "Un pedido no puede tener total $total" }
  }
}
```

Fíjate: cero `import` de frameworks.

### application: puertos y use case

El **puerto de entrada** dice qué puede pedirle el mundo al núcleo:

```kotlin
interface CrearPedido {
  fun ejecutar(clienteId: String, total: Double): Pedido
}
```

El **puerto de salida** dice qué necesita el núcleo del mundo:

```kotlin
interface PedidoRepository {
  fun guardar(pedido: Pedido)
}
```

El **use case** implementa el puerto de entrada usando el de salida:

```kotlin
class CrearPedidoService(
  private val repository: PedidoRepository,
) : CrearPedido {
  override fun ejecutar(clienteId: String, total: Double): Pedido {
    val pedido = Pedido(PedidoId(UUID.randomUUID().toString()), clienteId, total)
    repository.guardar(pedido)
    return pedido
  }
}
```

### infrastructure: los adapters

Adapter **secundario** (implementa el puerto de salida con Exposed):

```kotlin
object PedidosTable : Table("pedidos") {
  val id = varchar("id", 36)
  val clienteId = varchar("cliente_id", 36)
  val total = double("total")
  override val primaryKey = PrimaryKey(id)
}

class ExposedPedidoRepository : PedidoRepository {
  override fun guardar(pedido: Pedido) {
    transaction {
      PedidosTable.insert {
        it[id] = pedido.id.value
        it[clienteId] = pedido.clienteId
        it[total] = pedido.total
      }
    }
  }
}
```

Adapter **primario** (una ruta de Ktor que llama al puerto de entrada):

```kotlin
@Serializable
data class CrearPedidoRequest(val clienteId: String, val total: Double)

@Serializable
data class PedidoResponse(val id: String, val total: Double)

fun Route.pedidoRoutes(crearPedido: CrearPedido) {
  post("/api/v1/pedidos") {
    val request = call.receive<CrearPedidoRequest>()
    val pedido = crearPedido.ejecutar(request.clienteId, request.total)
    call.respond(HttpStatusCode.Created, PedidoResponse(pedido.id.value, pedido.total))
  }
}
```

## Los modelos no cruzan fronteras

Hay **tres** representaciones del mismo pedido, y eso es a propósito:

| Modelo | Vive en | Lo cambia |
| --- | --- | --- |
| `CrearPedidoRequest` / `PedidoResponse` | infrastructure (web) | El contrato de la API |
| `Pedido` | domain | Las reglas de negocio |
| `PedidosTable` | infrastructure (persistencia) | El esquema de la base de datos |

Si renombras una columna, solo cambia el adapter. Si cambias el JSON, solo cambia la ruta. El dominio no se entera.

Para traducir entre modelos, en Kotlin se usan **extension functions** dentro del adapter, en vez de clases `*Mapper`:

```kotlin
fun Pedido.toResponse() = PedidoResponse(id.value, total)

private fun ResultRow.toPedido() = Pedido(
  id = PedidoId(this[PedidosTable.id]),
  clienteId = this[PedidosTable.clienteId],
  total = this[PedidosTable.total],
)
```

## Por qué Ktor + Exposed encaja bien con hexagonal

Ktor y Exposed son librerías **livianas**: las llamas tú, no te imponen anotaciones en tus clases. Con frameworks más pesados es tentador poner `@Entity` o `@Serializable` en el modelo de dominio "para ahorrar código", y ahí el framework se mete en el centro.

Con esta pila, mantener el dominio limpio es el camino natural: el código de Ktor y Exposed vive solo en los adapters.

## Transacciones: un detalle que se escapa fácil

La transacción es un concepto de infraestructura, pero a veces el use case necesita que **varias** operaciones sean atómicas. Dos estrategias comunes:

1. **El adapter abre la transacción** (como arriba). Simple, pero cada llamada al puerto es su propia transacción.
2. **Un puerto `TransactionRunner`** que el use case usa para envolver varias operaciones, implementado con `transaction { }` en infraestructura.

Un truco útil dentro de los adapters: declarar las funciones de persistencia como extensiones de `Transaction`, así **solo se pueden llamar dentro de un bloque `transaction { }`**, y olvidarlo es un error de compilación:

```kotlin
private fun Transaction.insertarPedido(pedido: Pedido) {
  PedidosTable.insert { /* ... */ }
}
```

La plantilla [ktor-hexagonal-multimodule](https://github.com/mobiletoly/ktor-hexagonal-multimodule) persigue el mismo objetivo con una anotación propia, `@MustBeCalledInTransactionContext`.

-----

## Resumen en 5 líneas

1. El núcleo (domain + application) no conoce ninguna tecnología; la infraestructura se conecta por puertos.
2. Puerto de entrada = lo que el mundo puede pedirle al núcleo; puerto de salida = lo que el núcleo necesita del mundo.
3. Adapters primarios llaman al núcleo (rutas HTTP); adapters secundarios son llamados por él (repositorios, clientes).
4. Request/Response, dominio y tablas son modelos distintos; se traducen con extension functions en el adapter.
5. Ktor y Exposed son livianos, así que el framework queda en el borde sin esfuerzo extra.

-----

## Siguiente lección

[3. DDD y monolito modular](03-ddd-y-monolito-modular.md): qué hacer cuando el sistema crece y un solo hexágono no alcanza.
