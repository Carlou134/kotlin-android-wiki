# DDD y monolito modular

## En una frase

DDD (Domain-Driven Design) parte un sistema grande en **bounded contexts**, zonas del negocio con su propio modelo y vocabulario; un monolito modular los implementa como módulos separados dentro de una sola aplicación.

-----

## Antes de empezar

Conviene que ya sepas:

* Las capas y los puertos: [Arquitectura hexagonal](02-arquitectura-hexagonal.md).

Palabras nuevas (también están en el [Glosario](Glosario.md)):

* **Bounded context:** Una parte del negocio con límites claros, donde cada palabra significa una sola cosa.
* **Lenguaje ubicuo:** El vocabulario que comparten el código y la gente de negocio dentro de un bounded context.
* **Monolito modular:** Una sola aplicación desplegable, dividida internamente en módulos que no se pisan.
* **Shared kernel:** Un módulo chico con lo que varios contextos comparten de verdad (IDs, tipos base).
* **Anti-Corruption Layer (ACL):** Capa que traduce el modelo de un sistema externo al tuyo, para que el ajeno no "contamine" tu dominio.

-----

## El problema: la misma palabra, varios significados

En una tienda, la palabra **"cliente"** significa cosas distintas según quién la diga:

* Para **ventas**: alguien con un carrito y un historial de pedidos.
* Para **facturación**: un número fiscal, una dirección y un medio de pago.
* Para **identidad**: un email, una contraseña y unos roles.

Si haces **una sola** clase `Cliente` para todos, termina con 40 propiedades y cada equipo rompe lo del otro. Ese es el problema que resuelve DDD.

## Bounded contexts

La solución es aceptar que hay **varios modelos**, cada uno válido dentro de su límite:

```
┌─── pedidos ────────┐  ┌─── facturacion ────┐  ┌─── identidad ──────┐
│ Cliente(id,        │  │ Cliente(id,        │  │ Usuario(id, email, │
│   historial)       │  │   cuit, direccion) │  │   roles)           │
└────────────────────┘  └────────────────────┘  └────────────────────┘
          └──────────── comparten solo ClienteId ───────────┘
```

Cada contexto tiene **su propio hexágono**: su domain, su application y su infrastructure.

## Monolito modular con Gradle

No necesitas microservicios para separar contextos. En Gradle, cada contexto es un módulo:

```
settings.gradle.kts
  include(":shared")
  include(":pedidos")
  include(":facturacion")
  include(":identidad")
  include(":app")         // composition root: arma todo y arranca el servidor
```

Y cada módulo tiene sus capas como paquetes:

```
pedidos/src/main/kotlin/com/tienda/pedidos/
  domain/
  application/
  infrastructure/
```

La ganancia: si `:facturacion` no declara una dependencia de `:pedidos` en su `build.gradle.kts`, **no puede** importar sus clases. El límite lo hace cumplir el compilador.

## Dos formas de partir en módulos

Hay dos estrategias comunes, y ambas son hexagonales:

| | Por capa técnica | Por bounded context |
| --- | --- | --- |
| Módulos | `:core`, `:adapters`, `:infra` | `:pedidos`, `:facturacion`, `:identidad` |
| Capas | Cada capa es un módulo | Cada contexto tiene sus capas adentro |
| Qué protege el compilador | Que el núcleo no importe frameworks | Que un contexto no se meta en otro |
| Cuándo conviene | Proyectos chicos con un solo dominio | Sistemas con varias áreas de negocio |

La plantilla [ktor-hexagonal-multimodule](https://github.com/mobiletoly/ktor-hexagonal-multimodule) usa la primera: `core` (dominio + puertos), `adapters` (con `primary-web`, `persist` y `remoting`) e `infra` (arranque).

Para un sistema con varias áreas de negocio, la segunda escala mejor: cuando un contexto crece, lo puedes sacar a un servicio aparte con poco esfuerzo, porque ya está aislado. Las reglas de capas *dentro* de cada contexto se protegen con tests de arquitectura (lección 6).

## Shared kernel: lo mínimo compartido

Algunas cosas sí son comunes a todos: un `ClienteId`, un tipo `Dinero`, utilidades de fecha. Van a `:shared`.

```kotlin
// :shared
@JvmInline
value class ClienteId(val value: String)
```

**Cuidado:** el shared kernel tiende a convertirse en un cajón de sastre. Regla práctica: si algo lo usa un solo contexto, no va en `:shared`.

## Cómo se hablan los contextos

Un contexto **nunca** accede a las tablas ni a las clases internas de otro. Se comunica por su **puerto de entrada**:

```kotlin
// En :facturacion, un puerto de salida propio
interface ConsultarPedidos {
  fun totalDe(pedidoId: String): Double
}

// En :app (composition root), el adapter que lo conecta con :pedidos
class ConsultarPedidosAdapter(
  private val obtenerPedido: ObtenerPedido, // puerto de entrada de :pedidos
) : ConsultarPedidos {
  override fun totalDe(pedidoId: String) = obtenerPedido.ejecutar(pedidoId).total
}
```

`:facturacion` define lo que necesita con *su* vocabulario. No sabe que existe `:pedidos`.

## Anti-Corruption Layer: proteger tu dominio de terceros

Cuando integras una API externa, su modelo suele ser raro: nombres en otro idioma, estados codificados como números, campos que no te importan. Si usas ese modelo en tu dominio, el tercero **te corrompe**.

La ACL es un adapter que traduce:

```kotlin
// Lo que devuelve la API del proveedor de envíos (su modelo)
@Serializable
data class ShipmentDto(val trk: String, val st: Int, val eta_ts: Long)

// Tu modelo de dominio
enum class EstadoEnvio { PREPARANDO, EN_CAMINO, ENTREGADO }
data class Envio(val codigo: String, val estado: EstadoEnvio, val llegada: Instant)

// La ACL: el único lugar que conoce ambos
fun ShipmentDto.toDomain() = Envio(
  codigo = trk,
  estado = when (st) {
    0 -> EstadoEnvio.PREPARANDO
    1 -> EstadoEnvio.EN_CAMINO
    2 -> EstadoEnvio.ENTREGADO
    else -> error("Estado de envío desconocido: $st")
  },
  llegada = Instant.fromEpochMilliseconds(eta_ts),
)
```

Si integras **varios** proveedores, cada uno tiene su propia ACL, y el dominio ve un solo modelo `Envio`.

-----

## Resumen en 5 líneas

1. Un bounded context es una zona del negocio con su propio modelo; la misma palabra puede significar cosas distintas en cada uno.
2. Un monolito modular implementa cada contexto como un módulo de Gradle, y el compilador vigila los límites.
3. Se puede partir por capa técnica (`core`/`adapters`/`infra`) o por contexto; la segunda escala mejor con varias áreas de negocio.
4. El shared kernel guarda solo lo realmente común; los contextos se hablan por puertos, nunca por tablas.
5. Una Anti-Corruption Layer traduce el modelo de un tercero al tuyo para que no contamine el dominio.

-----

## Siguiente lección

[4. Composition root](04-composition-root.md): dónde se conectan todas estas piezas.
