# Inyección de dependencias con Koin

## En una frase

Koin es un contenedor de inyección de dependencias para Kotlin: describes en **módulos** cómo se crea cada pieza, y Koin las construye y las conecta al arrancar, sin generar código ni usar reflexión pesada.

-----

## Antes de empezar

Conviene que ya sepas:

* Qué es el composition root y por qué no usar el contenedor desde el núcleo: [Composition root](../12-arquitectura/04-composition-root.md).
* Organizar las rutas: [Rutas y autenticación por ruta](03-rutas-y-autenticacion.md).

Palabras nuevas (también están en el [Glosario](Glosario.md)):

* **Definición:** La regla que le dice a Koin cómo crear un tipo (`single { ... }`).
* **`single`:** Una sola instancia para toda la aplicación.
* **`factory`:** Una instancia nueva cada vez que se pide.
* **Qualifier:** Un nombre que distingue dos definiciones del mismo tipo.
* **Grafo de dependencias:** El conjunto de piezas y quién necesita a quién.

-----

## Lo básico

```kotlin
val pedidosModule = module {
  single<PedidoRepository> { ExposedPedidoRepository(get()) }
  single<CrearPedido> { CrearPedidoService(get(), get()) }
}
```

* `single<Interfaz> { Implementacion(...) }` registra la implementación **bajo el tipo de la interfaz**. Quien pide `PedidoRepository` recibe un `ExposedPedidoRepository`, sin saberlo.
* `get()` le pide a Koin la dependencia del tipo que el constructor espera.

La forma corta, con referencias a constructores:

```kotlin
val pedidosModule = module {
  singleOf(::ExposedPedidoRepository) bind PedidoRepository::class
  singleOf(::CrearPedidoService) bind CrearPedido::class
}
```

`singleOf` resuelve todos los parámetros del constructor con `get()` automáticamente, y `bind` registra la interfaz.

## `single` vs `factory`

| | `single` | `factory` |
| --- | --- | --- |
| Instancias | Una para toda la app | Una nueva en cada `get()` |
| Úsalo para | Servicios sin estado, repositorios, clientes HTTP, pools | Objetos con estado de corta vida |

En un backend, casi todo es `single`: los use cases y los repositorios no guardan estado entre peticiones (el estado vive en la base). Si un use case **necesita** ser `factory` porque guarda estado, revisa el diseño: probablemente ese estado debería ser una variable local.

## Qualifiers: dos del mismo tipo

Cuando tienes dos `Database` (una para el schema principal y otra para `auth`):

```kotlin
val infraModule = module {
  single(named("principal")) { Database.connect(crearDataSource(get<DbConfig>().principal)) }
  single(named("auth")) { Database.connect(crearDataSource(get<DbConfig>().auth)) }
}

val authModule = module {
  single<UsuarioRepository> { ExposedUsuarioRepository(get(named("auth"))) }
}
```

Úsalos con moderación: si hay muchos qualifiers, suele ser mejor un tipo propio (`class AuthDatabase(val db: Database)`), que el compilador sí verifica.

## Un módulo por contexto

La convención (la misma que usa la plantilla [ktor-hexagonal-multimodule](https://github.com/mobiletoly/ktor-hexagonal-multimodule)): cada módulo de Gradle expone **su** módulo de Koin, en `infrastructure/di/`:

```
pedidos/src/main/kotlin/com/tienda/pedidos/infrastructure/di/PedidosModule.kt
facturacion/src/main/kotlin/com/tienda/facturacion/infrastructure/di/FacturacionModule.kt
```

Cada contexto sabe cómo armar **sus** piezas; nadie más conoce sus implementaciones. El composition root solo los junta:

```kotlin
fun Application.configurarDependencias() {
  install(Koin) {
    slf4jLogger()
    modules(
      configModule(environment.config),
      infraModule,
      pedidosModule,
      facturacionModule,
    )
  }
}
```

La configuración entra como un módulo más, así ningún otro módulo lee variables de entorno por su cuenta:

```kotlin
fun configModule(config: ApplicationConfig) = module {
  single { DbConfig.from(config) }
  single { JwtConfig.from(config) }
}
```

## Koin en las rutas

Dentro de Ktor, Koin agrega `inject()` y `get()` a `Application` y `Route`:

```kotlin
fun Application.configurarRutas() {
  val crearPedido by inject<CrearPedido>()
  val cancelarPedido by inject<CancelarPedido>()

  routing {
    route("/api/v1") {
      authenticate("usuario") {
        pedidoRoutes(crearPedido, cancelarPedido)
      }
    }
  }
}
```

Fíjate en que `pedidoRoutes` **recibe** los puertos por parámetro: no sabe que existe Koin. Así puedes testear las rutas pasándoles implementaciones falsas, sin contenedor.

## Dónde **no** usar Koin

* **En el dominio y en la capa application:** todo entra por constructor. Nada de `KoinComponent` ni `by inject()`. Ya lo viste en [Composition root](../12-arquitectura/04-composition-root.md): eso es un service locator.
* **En los tests unitarios:** construye la clase a mano con sus dependencias falsas. Es más simple y más rápido que levantar Koin.

Una buena señal de diseño: si borraras Koin, lo único que tendrías que reescribir son los archivos `di/` y el composition root.

## Verificar el grafo

El peor error de Koin es en tiempo de ejecución: falta una definición y la app explota al arrancar (o peor, la primera vez que alguien pide ese tipo). Koin puede verificar el grafo en un test:

```kotlin
class KoinModulesTest : FunSpec({
  test("todos los módulos se pueden resolver") {
    appModules.verify(extraTypes = listOf(ApplicationConfig::class))
  }
})
```

`verify()` revisa que cada constructor tenga sus dependencias declaradas, sin crear nada. `extraTypes` lista los tipos que vienen de afuera (por ejemplo, la configuración de Ktor).

## Alternativas

* **A mano:** para un proyecto chico, el composition root con constructores explícitos es perfectamente válido, y te da errores en tiempo de compilación.
* **Plugin de DI de Ktor:** desde Ktor 3.2 hay un plugin de inyección de dependencias propio, integrado con la configuración de Ktor.
* **Hilt / Dagger:** generan código y verifican el grafo al compilar; son el estándar en Android, menos comunes en servidores Ktor.

Koin se elige por simplicidad: es solo Kotlin, sin anotaciones ni generación de código.

-----

## Resumen en 5 líneas

1. Koin describe en módulos cómo crear cada tipo; `single<Interfaz> { Impl(get()) }` o `singleOf(::Impl) bind Interfaz::class`.
2. En un backend casi todo es `single`; los qualifiers distinguen dos definiciones del mismo tipo, pero un tipo propio es más seguro.
3. Un módulo de Koin por módulo de Gradle, en `infrastructure/di/`, y el composition root los instala todos con `install(Koin)`.
4. Las rutas reciben los puertos por parámetro; el núcleo nunca usa Koin, y los tests unitarios tampoco.
5. Verifica el grafo con `verify()` en un test, para no descubrir una definición faltante en producción.

-----

## Siguiente lección

Terminaste Ktor y Koin. Vuelve al índice: [README principal](../README.md).
