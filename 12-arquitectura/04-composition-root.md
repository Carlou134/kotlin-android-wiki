# Composition root

## En una frase

El composition root es el **único** lugar de la aplicación que conoce todas las implementaciones concretas y las conecta entre sí; el resto del código solo conoce interfaces.

-----

## Antes de empezar

Conviene que ya sepas:

* Qué son los puertos y adapters: [Arquitectura hexagonal](02-arquitectura-hexagonal.md).
* Cómo se separan los módulos: [DDD y monolito modular](03-ddd-y-monolito-modular.md).

Palabras nuevas (también están en el [Glosario](Glosario.md)):

* **Composition root:** Punto de arranque donde se crean y conectan las piezas concretas.
* **Inyección de dependencias (DI):** Pasarle a una clase lo que necesita desde afuera, en vez de que lo cree ella.
* **Contenedor de DI:** Librería que crea los objetos y resuelve sus dependencias por ti (Koin, Hilt).
* **Koin:** Contenedor de DI liviano para Kotlin, sin generación de código.

-----

## El problema

Si el use case usa una interfaz, alguien tiene que decidir **qué implementación** recibe. ¿Quién?

Si lo decide el propio use case, volviste al acoplamiento:

```kotlin
class CrearPedidoService : CrearPedido {
  private val repository = ExposedPedidoRepository() // ❌ conoce el detalle
}
```

La respuesta: lo decide **un solo lugar**, en el borde de la aplicación, al arrancar.

## A mano, sin librerías

Antes de cualquier framework, la idea es esta:

```kotlin
fun main() {
  // 1. Crear los detalles
  val repository: PedidoRepository = ExposedPedidoRepository()

  // 2. Crear los use cases inyectando los detalles
  val crearPedido: CrearPedido = CrearPedidoService(repository)

  // 3. Arrancar el servidor pasando los puertos a las rutas
  embeddedServer(Netty, port = 8080) {
    routing { pedidoRoutes(crearPedido) }
  }.start(wait = true)
}
```

Eso **es** un composition root. Con tres clases alcanza. Con cien, se vuelve tedioso, y ahí entra un contenedor.

## Con Koin

Koin describe "cómo se crea cada cosa" en **módulos**, y la convención es tener **un módulo de Koin por módulo de Gradle**:

```kotlin
// En :pedidos — infrastructure/di/PedidosModule.kt
val pedidosModule = module {
  single<PedidoRepository> { ExposedPedidoRepository() }
  single<CrearPedido> { CrearPedidoService(get()) }
}
```

* `single { }` crea una sola instancia para toda la app.
* `get()` le pide a Koin la dependencia del tipo que corresponde.
* `single<CrearPedido>` registra la implementación **bajo el tipo de la interfaz**.

Y el composition root solo los junta:

```kotlin
// En :app — Application.kt
fun Application.module() {
  install(Koin) {
    modules(pedidosModule, facturacionModule, identidadModule)
  }

  val crearPedido by inject<CrearPedido>()

  routing {
    pedidoRoutes(crearPedido)
  }
}
```

## Por qué las rutas viven en el composition root

Una decisión común en monolitos modulares: los módulos de negocio son **librerías** (no arrancan nada) y solo `:app` instala plugins y registra rutas.

Ventajas:

* Una sola vista de **todas** las rutas expuestas, con su autenticación.
* Los módulos de negocio se pueden testear sin levantar un servidor.
* Si mañana sacas un contexto a otro servicio, solo cambias el composition root.

## Composition root ≠ service locator

Hay una trampa: usar el contenedor **desde adentro** del código de negocio.

```kotlin
class CrearPedidoService : CrearPedido, KoinComponent {
  private val repository: PedidoRepository by inject() // ❌ service locator
}
```

Funciona, pero ahora el use case depende de Koin y esconde sus dependencias: mirando el constructor no sabes qué necesita. Regla: **en el núcleo, todo entra por constructor**. Koin solo aparece en los archivos `di/` y en el composition root.

-----

## Resumen en 5 líneas

1. El composition root es el único lugar que conoce las implementaciones concretas y las conecta.
2. Se puede hacer a mano en `main()`; un contenedor como Koin lo automatiza cuando hay muchas piezas.
3. Convención: un módulo de Koin por módulo de Gradle, y el composition root los instala todos.
4. Los módulos de negocio son librerías; las rutas y los plugins se registran solo en `:app`.
5. Nunca uses el contenedor desde el núcleo: todo entra por constructor.

-----

## Siguiente lección

[5. La capa application](05-capa-application.md): cómo diseñar los use cases.
