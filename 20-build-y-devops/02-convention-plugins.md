# Convention plugins y composite builds

## En una frase

Un convention plugin es un plugin de Gradle **propio** que guarda la configuración común a muchos módulos (versión de Java, ktlint, detekt, Kover, tests); vive en un build aparte llamado `build-logic`, que se incluye en el proyecto como composite build, y cada módulo lo aplica con una línea.

-----

## Antes de empezar

Conviene que ya sepas:

* Los archivos de Gradle y el version catalog: [Gradle con Kotlin DSL y version catalog](01-gradle-y-version-catalog.md).
* Las herramientas de calidad: [Calidad: Kover, ktlint y detekt](../19-testing/05-calidad.md).

Palabras nuevas (también están en el [Glosario](Glosario.md)):

* **Convention plugin:** Plugin de Gradle propio que aplica una configuración común.
* **Precompiled script plugin:** Un archivo `.gradle.kts` dentro de `build-logic` que Gradle convierte en un plugin.
* **`build-logic`:** Carpeta con un build de Gradle independiente que contiene los convention plugins.
* **Composite build:** Un build que incluye a otro build completo con `includeBuild`.

-----

## El problema: copiar y pegar la configuración

Con 8 módulos, cada `build.gradle.kts` repite lo mismo:

```kotlin
plugins {
  alias(libs.plugins.kotlin.jvm)
  alias(libs.plugins.ktlint)
  alias(libs.plugins.detekt)
  alias(libs.plugins.kover)
}
kotlin { jvmToolchain(21) }
tasks.test { useJUnitPlatform() }
detekt { buildUponDefaultConfig = true; config.setFrom("$rootDir/config/detekt/detekt.yml") }
dependencies {
  testImplementation(libs.kotest.runner)
  testImplementation(libs.mockk)
}
```

Cambiar la configuración de detekt es editar 8 archivos, y uno siempre queda distinto.

La solución vieja era configurar todo desde la raíz con `subprojects { ... }`. Funciona, pero acopla todos los módulos a la raíz, aplica cosas a módulos que no las necesitan y hace que el build sea difícil de entender ("¿de dónde salió esta configuración?"). Gradle hoy recomienda **convention plugins**.

## La estructura

```
tienda/
├── settings.gradle.kts
├── gradle/libs.versions.toml
├── build-logic/
│   ├── settings.gradle.kts
│   ├── build.gradle.kts
│   └── src/main/kotlin/
│       ├── tienda.kotlin-conventions.gradle.kts
│       ├── tienda.library-conventions.gradle.kts
│       └── tienda.ktor-app-conventions.gradle.kts
├── pedidos/build.gradle.kts
└── app/build.gradle.kts
```

### Incluir `build-logic` en el proyecto

`settings.gradle.kts` de la raíz:

```kotlin
pluginManagement {
  includeBuild("build-logic")   // composite build: los plugins de build-logic quedan disponibles
}

rootProject.name = "tienda"
include(":shared", ":pedidos", ":app")
```

### El build de `build-logic`

`build-logic/settings.gradle.kts`, que reutiliza el **mismo** version catalog:

```kotlin
dependencyResolutionManagement {
  versionCatalogs {
    create("libs") {
      from(files("../gradle/libs.versions.toml"))
    }
  }
}

rootProject.name = "build-logic"
```

`build-logic/build.gradle.kts`:

```kotlin
plugins {
  `kotlin-dsl`   // permite escribir plugins como archivos .gradle.kts
}

dependencies {
  // Los plugins que tus convention plugins van a aplicar, como dependencias normales
  implementation(libs.kotlin.gradle.plugin)
  implementation(libs.ktlint.gradle.plugin)
  implementation(libs.detekt.gradle.plugin)
  implementation(libs.kover.gradle.plugin)
}
```

## Un convention plugin

`build-logic/src/main/kotlin/tienda.kotlin-conventions.gradle.kts`:

```kotlin
plugins {
  id("org.jetbrains.kotlin.jvm")
  id("org.jlleitschuh.gradle.ktlint")
  id("io.gitlab.arturbosch.detekt")
  id("org.jetbrains.kotlinx.kover")
}

kotlin {
  jvmToolchain(21)
}

tasks.test {
  useJUnitPlatform()
}

detekt {
  buildUponDefaultConfig = true
  config.setFrom("$rootDir/config/detekt/detekt.yml")
}

val libs = the<VersionCatalogsExtension>().named("libs")

dependencies {
  "testImplementation"(libs.findLibrary("kotest-runner").get())
  "testImplementation"(libs.findLibrary("mockk").get())
}
```

* El **nombre del archivo** es el ID del plugin: `tienda.kotlin-conventions`.
* Dentro de un convention plugin, el acceso tipado `libs.kotest.runner` no está disponible directamente; se usa `VersionCatalogsExtension` con `findLibrary`.

Los plugins se pueden **componer**: uno para librerías y otro para las aplicaciones Ktor, ambos sobre la base común.

```kotlin
// tienda.ktor-app-conventions.gradle.kts
plugins {
  id("tienda.kotlin-conventions")
  id("io.ktor.plugin")
  application
}
```

## Cómo queda un módulo

```kotlin
// pedidos/build.gradle.kts
plugins {
  id("tienda.library-conventions")
}

dependencies {
  implementation(project(":shared"))
  implementation(libs.exposed.core)
}
```

El módulo solo declara **lo que lo hace distinto**: sus dependencias. Todo lo común viene del convention plugin. Cambiar la versión de Java o la configuración de detekt es editar **un** archivo.

## Composite builds en general

`includeBuild` sirve para más que `build-logic`. Incluye **otro build completo** como si fuera parte del tuyo. El otro uso típico: desarrollar una librería propia y probarla en una aplicación al mismo tiempo, sin publicarla.

```kotlin
// settings.gradle.kts de la aplicación
includeBuild("../mi-libreria")   // Gradle usa el código local en vez de la versión publicada
```

## Cuándo crear un convention plugin nuevo

* Cuando **3 o más** módulos repiten la misma configuración.
* Cuando hay **tipos** de módulos distintos (librería, aplicación Ktor, módulo de tests de arquitectura) con configuraciones propias.

Y cuándo no: para algo que usa un solo módulo. Ahí va en el `build.gradle.kts` de ese módulo.

-----

## Resumen en 5 líneas

1. Un convention plugin guarda la configuración común (Java, ktlint, detekt, Kover, tests) para no copiarla en cada módulo.
2. Vive en `build-logic`, un build independiente incluido con `pluginManagement { includeBuild("build-logic") }`.
3. Cada archivo `nombre.gradle.kts` en `build-logic/src/main/kotlin` es un plugin con ese ID; se pueden componer.
4. `build-logic` reutiliza el mismo version catalog; dentro de los plugins se accede con `VersionCatalogsExtension`.
5. Un módulo aplica su plugin con una línea y solo declara sus dependencias; `includeBuild` también sirve para desarrollar librerías locales.

-----

## Siguiente lección

[3. Docker y docker-compose](03-docker-y-docker-compose.md): el entorno completo, en un archivo.
