# Gradle con Kotlin DSL y version catalog

## En una frase

Gradle es la herramienta que compila, testea y empaqueta tu proyecto: lo describes con archivos `.gradle.kts` (Kotlin), divides el proyecto en módulos, y centralizas las versiones de todas las dependencias en un **version catalog** (`libs.versions.toml`).

-----

## Antes de empezar

Conviene que ya sepas:

* Por qué el proyecto se divide en módulos: [DDD y monolito modular](../12-arquitectura/03-ddd-y-monolito-modular.md).
* Lambdas con receptor, porque los archivos de Gradle son un DSL: [Funciones de orden superior](../09-funcional/01-funciones-de-orden-superior.md).

Palabras nuevas (también están en el [Glosario](Glosario.md)):

* **Gradle:** Herramienta de build: compila, corre tests, empaqueta y resuelve dependencias.
* **Tarea (task):** Una unidad de trabajo de Gradle: `compileKotlin`, `test`, `build`.
* **Módulo (subproyecto):** Una parte del proyecto con su propio `build.gradle.kts`.
* **Dependencia transitiva:** Una dependencia de tu dependencia.
* **Gradle wrapper:** Script (`gradlew`) que descarga y usa la versión exacta de Gradle del proyecto.
* **Toolchain:** La versión de JDK con la que Gradle compila, independiente de la que tengas instalada.
* **Version catalog:** Archivo central (`libs.versions.toml`) con las versiones y coordenadas de las dependencias.

-----

## Los archivos de un proyecto

```
tienda/
├── settings.gradle.kts          ← qué módulos tiene el proyecto
├── build.gradle.kts             ← configuración de la raíz (mínima)
├── gradle/
│   ├── libs.versions.toml       ← el version catalog
│   └── wrapper/
│       └── gradle-wrapper.properties
├── gradlew  /  gradlew.bat      ← el wrapper
├── shared/build.gradle.kts
├── pedidos/build.gradle.kts
└── app/build.gradle.kts
```

### `settings.gradle.kts`

```kotlin
rootProject.name = "tienda"

include(":shared")
include(":pedidos")
include(":facturacion")
include(":app")
```

Cada `include` es un módulo, con su carpeta y su `build.gradle.kts`.

### `build.gradle.kts` de un módulo

```kotlin
plugins {
  alias(libs.plugins.kotlin.jvm)
  alias(libs.plugins.kotlin.serialization)
}

dependencies {
  implementation(project(":shared"))
  implementation(libs.exposed.core)
  implementation(libs.kotlinx.coroutines.core)

  testImplementation(libs.kotest.runner)
  testImplementation(libs.mockk)
}

kotlin {
  jvmToolchain(21)
}

tasks.test {
  useJUnitPlatform()
}
```

## Configuraciones de dependencias

| Configuración | Disponible en | ¿La ven los módulos que dependen de este? |
| --- | --- | --- |
| `implementation` | Compilación y ejecución | No |
| `api` | Compilación y ejecución | **Sí** (se vuelve transitiva) |
| `compileOnly` | Solo compilación | No |
| `runtimeOnly` | Solo ejecución | No |
| `testImplementation` | Solo los tests | No |

**Usa `implementation` por defecto.** Si `:pedidos` usa Exposed con `implementation`, el módulo `:app` que depende de `:pedidos` **no** puede importar Exposed sin declararlo. Eso protege los límites entre módulos: los detalles de un módulo no se filtran a los demás.

`api` es para cuando los tipos de esa dependencia aparecen en tu **API pública**. Por ejemplo, si `:shared` expone `ClienteId` y usa tipos de kotlinx-datetime en sus firmas, los que lo usan necesitan esa librería.

`runtimeOnly` es típico para drivers e implementaciones: el driver de PostgreSQL o Logback se necesitan al ejecutar, pero tu código nunca los importa.

## El wrapper: la misma versión para todos

```
./gradlew build          # en Linux, Mac y Git Bash
gradlew.bat build        # en Windows (cmd)
```

`gradlew` descarga la versión de Gradle definida en `gradle-wrapper.properties` y la usa. Tú, tu compañero y el CI usan **la misma**. Los archivos del wrapper se suben al repositorio; nunca uses un `gradle` instalado globalmente.

## Toolchain: la misma versión de Java

```kotlin
kotlin {
  jvmToolchain(21)
}
```

Gradle compila con JDK 21 aunque tengas otra versión instalada (si no la encuentra, puede descargarla). Así el proyecto no depende de qué Java tenga cada uno en su máquina.

## El version catalog

Sin catálogo, cada módulo escribe sus versiones a mano. Con 10 módulos, un día uno queda con Ktor 3.0 y otro con Ktor 3.1, y aparecen errores raros.

`gradle/libs.versions.toml`:

```toml
[versions]
kotlin = "2.1.0"
ktor = "3.1.0"
exposed = "0.58.0"
kotest = "5.9.1"

[libraries]
ktor-server-core = { module = "io.ktor:ktor-server-core", version.ref = "ktor" }
ktor-server-netty = { module = "io.ktor:ktor-server-netty", version.ref = "ktor" }
exposed-core = { module = "org.jetbrains.exposed:exposed-core", version.ref = "exposed" }
kotest-runner = { module = "io.kotest:kotest-runner-junit5", version.ref = "kotest" }

[bundles]
ktor-server = ["ktor-server-core", "ktor-server-netty"]

[plugins]
kotlin-jvm = { id = "org.jetbrains.kotlin.jvm", version.ref = "kotlin" }
kotlin-serialization = { id = "org.jetbrains.kotlin.plugin.serialization", version.ref = "kotlin" }
```

(Las versiones son de ejemplo; usa las del proyecto.)

* **`[versions]`:** una versión, usada por varias librerías (todas las de Ktor comparten `ktor`).
* **`[libraries]`:** cada dependencia con un alias. Los guiones se vuelven puntos: `ktor-server-core` se usa como `libs.ktor.server.core`.
* **`[bundles]`:** grupos de librerías que siempre van juntas: `implementation(libs.bundles.ktor.server)`.
* **`[plugins]`:** los plugins de Gradle: `alias(libs.plugins.kotlin.jvm)`.

Actualizar Ktor es cambiar **una** línea, y todos los módulos se actualizan juntos. El IDE además autocompleta `libs.`.

## Tareas que vas a usar

```
./gradlew build                  # compila, testea y empaqueta todo
./gradlew test                   # todos los tests
./gradlew :pedidos:test          # solo los tests de un módulo
./gradlew :pedidos:test --tests "*CancelarPedido*"   # solo algunos tests
./gradlew check                  # tests + verificaciones (ktlint, detekt, kover, si están configurados)
./gradlew :app:dependencies      # el árbol de dependencias, para encontrar conflictos de versiones
./gradlew build --continue       # no corta en el primer error: muestra todos los fallos
./gradlew tasks                  # qué tareas existen
```

Gradle es **incremental**: si un módulo no cambió, no lo recompila ni vuelve a correr sus tests. Por eso `./gradlew test` la segunda vez tarda mucho menos. Si algo se comporta raro, `./gradlew clean` borra lo generado y empieza de cero.

-----

## Resumen en 5 líneas

1. `settings.gradle.kts` declara los módulos; cada módulo tiene su `build.gradle.kts` con plugins y dependencias.
2. `implementation` por defecto (no filtra la dependencia a otros módulos); `api` solo si aparece en tu API pública; `runtimeOnly` para drivers.
3. El wrapper (`./gradlew`) y el toolchain (`jvmToolchain(21)`) garantizan la misma versión de Gradle y de Java para todos.
4. El version catalog (`libs.versions.toml`) centraliza versiones, librerías, bundles y plugins: actualizar es cambiar una línea.
5. `:modulo:tarea` y `--tests` para correr solo lo que tocaste; Gradle es incremental y `clean` vuelve a cero.

-----

## Siguiente lección

[2. Convention plugins y composite builds](02-convention-plugins.md): no repetir la configuración en cada módulo.
