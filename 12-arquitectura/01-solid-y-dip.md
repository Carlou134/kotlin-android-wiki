# SOLID e inversión de dependencias

## En una frase

SOLID son cinco principios para que el código se pueda cambiar sin romper otras partes; el más importante para la arquitectura es la **inversión de dependencias**: el código importante no depende de los detalles, los detalles dependen de él.

-----

## Antes de empezar

Conviene que ya sepas:

* Crear clases y pasarles datos por constructor: [Clases](../06-poo/01-clases.md).
* Qué es una interfaz (se ve en `08-poo-avanzada`): un contrato con funciones sin implementar.

Palabras nuevas (también están en el [Glosario](Glosario.md)):

* **Acoplamiento:** Cuánto depende una pieza de otra. Mucho acoplamiento = si cambias una, se rompe la otra.
* **Abstracción:** Una interfaz que describe *qué* se hace sin decir *cómo*.
* **Detalle:** La implementación concreta: la base de datos, el framework web, el cliente HTTP.
* **Inversión de dependencias (DIP):** Principio por el que lo importante define una interfaz y lo concreto la implementa.

-----

## El problema

Imaginá un servicio que crea pedidos y los guarda directamente con la base de datos:

```kotlin
class CrearPedidoService {
  fun crear(clienteId: String, total: Double) {
    if (total <= 0) throw IllegalArgumentException("Total inválido")

    // Lógica de negocio mezclada con SQL
    val conexion = DriverManager.getConnection("jdbc:postgresql://localhost/tienda")
    conexion.prepareStatement("INSERT INTO pedidos ...").executeUpdate()
  }
}
```

¿Qué pasa si mañana cambias de base de datos? Tocas la regla de negocio.

¿Y si quieres testear que un total negativo se rechaza? Necesitas una base de datos levantada.

La regla de negocio ("un pedido no puede tener total cero o negativo") quedó **atada** a un detalle técnico. Eso es acoplamiento.

## Los cinco principios en corto

| Letra | Principio | Idea en una línea |
| --- | --- | --- |
| **S** | Responsabilidad única | Una clase tiene un solo motivo para cambiar. |
| **O** | Abierto/cerrado | Agregas comportamiento nuevo sin modificar el código que ya funciona. |
| **L** | Sustitución de Liskov | Si `B` implementa `A`, puedes usar `B` donde se espera `A` sin sorpresas. |
| **I** | Segregación de interfaces | Interfaces chicas y específicas, no una gigante que obliga a implementar todo. |
| **D** | Inversión de dependencias | El código de alto nivel y el de bajo nivel dependen de una abstracción. |

Las cuatro primeras te ayudan a escribir buenas clases. La quinta te da la **arquitectura**.

## Inversión de dependencias, paso a paso

### 1. El negocio define lo que necesita

El servicio no dice "necesito PostgreSQL". Dice "necesito *algo* que guarde pedidos":

```kotlin
interface PedidoRepository {
  fun guardar(pedido: Pedido)
}
```

### 2. El negocio usa la interfaz

```kotlin
class CrearPedidoService(
  private val repository: PedidoRepository
) {
  fun crear(pedido: Pedido) {
    require(pedido.total > 0) { "Total inválido" }
    repository.guardar(pedido)
  }
}
```

`CrearPedidoService` no tiene ni un `import` de base de datos.

### 3. El detalle implementa la interfaz

```kotlin
class PostgresPedidoRepository : PedidoRepository {
  override fun guardar(pedido: Pedido) {
    // SQL real
  }
}
```

### ¿Qué se "invirtió"?

Sin DIP, la flecha de dependencia va así:

```
CrearPedidoService  ->  PostgreSQL
```

Con DIP, la flecha se da vuelta:

```
CrearPedidoService  ->  PedidoRepository  <-  PostgresPedidoRepository
```

Ahora el **detalle depende del negocio**, y no al revés. La interfaz pertenece al negocio: la escribe quien la necesita, no quien la implementa.

## La ganancia concreta: testear sin base de datos

```kotlin
class PedidoRepositoryEnMemoria : PedidoRepository {
  val guardados = mutableListOf<Pedido>()
  override fun guardar(pedido: Pedido) { guardados.add(pedido) }
}

fun main() {
  val repo = PedidoRepositoryEnMemoria()
  val service = CrearPedidoService(repo)

  service.crear(Pedido(id = "p-1", total = 100.0))
  println(repo.guardados.size) // 1
}
```

El test corre en milisegundos y prueba la regla de negocio sin nada externo.

## Ideas equivocadas comunes

* **"DIP es lo mismo que inyección de dependencias."** No. La inyección (pasar las cosas por constructor) es la *técnica*. DIP es el *principio* de hacia dónde apuntan las flechas. Puedes inyectar una clase concreta y seguir violando DIP.
* **"Hay que crear una interfaz para todo."** No. Creas una interfaz en la **frontera** entre el negocio y un detalle técnico. Una clase de utilidad interna no necesita una.

-----

## Resumen en 5 líneas

1. SOLID son cinco principios para que cambiar una parte no rompa otra.
2. La inversión de dependencias dice que el negocio define interfaces y los detalles las implementan.
3. La interfaz pertenece a quien la usa, no a quien la implementa.
4. Así la flecha de dependencia apunta hacia el negocio y no hacia la tecnología.
5. La ganancia inmediata: puedes testear las reglas con implementaciones en memoria.

-----

## Siguiente lección

[2. Arquitectura hexagonal](02-arquitectura-hexagonal.md): llevar esta idea a todo el proyecto.
