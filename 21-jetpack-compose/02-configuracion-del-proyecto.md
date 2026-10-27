# Configuración del proyecto

## En una frase

Antes de escribir un solo composable, el proyecto tiene que tener **versiones alineadas**: Kotlin 2.x, el plugin **Compose Compiler** con la misma versión de Kotlin y **KSP** compatible con ambos; si no, Android Studio ni siquiera abre el proyecto.

-----

## Antes de empezar

Conviene que ya sepas:

* Cómo funciona el version catalog y los bloques `plugins`: [Gradle con Kotlin DSL y version catalog](../20-build-y-devops/01-gradle-y-version-catalog.md).
* Clonar un repositorio con Git: [Git y merge requests](../20-build-y-devops/04-git-y-merge-requests.md).

Palabras nuevas (también están en el [Glosario](Glosario.md)):

* **AGP (Android Gradle Plugin):** El plugin de Gradle que sabe compilar apps Android. Cada versión de Android Studio soporta un rango de AGP.
* **Compose Compiler:** Plugin del compilador de Kotlin que transforma las funciones `@Composable`. Desde Kotlin 2.0 vive dentro del repositorio de Kotlin.
* **KSP (Kotlin Symbol Processing):** Procesador de anotaciones que genera código al compilar. Lo usan Room y Hilt.
* **Gradle Sync:** Cuando Android Studio lee los archivos de Gradle y arma el modelo del proyecto.
* **Emulador:** Un dispositivo Android virtual que corre en tu PC.

-----

## Dos caminos para empezar

### Camino A: clonar el repositorio del curso

```bash
cd C:\Aprendizaje autodidacta\kotlin-practice
git clone <url-del-repositorio>
```

En Android Studio: **Open** → selecciona la carpeta `todo_app`. Gradle sincroniza; sabes que terminó cuando la barra de progreso de abajo a la derecha desaparece y ves la estructura de carpetas a la izquierda.

El repositorio trae carpetas ya creadas:

* `navigation/` con un `NavigationRoot` vacío que se completa más adelante.
* `ui/theme/` con `Color.kt`, `Theme.kt` y `Type.kt`.
* `MainActivity`, la primera pantalla y punto de partida de los composables.

### Camino B: crear el proyecto desde cero (recomendado si el repo no abre)

> **Experiencia real:** el proyecto del curso usaba AGP 8.6 y Kotlin 1.9.24, y un Android Studio actual lo rechazó como "no compatible". Un proyecto viejo puede quedar fuera del rango que soporta tu Android Studio. La salida más limpia fue crear uno nuevo: la plantilla trae versiones que TU Android Studio garantiza.

1. **File → New → New Project → Empty Activity** (la de Compose; *Empty Views Activity* es para XML).
2. Configura:
   * **Name:** `Todo App`
   * **Package name:** el que quieras (por ejemplo `com.example.todo_app`). Si copias código del curso, ajusta la línea `package` y los imports.
   * **Minimum SDK:** API 26
   * **Build configuration language:** Kotlin DSL + Version catalog
3. **Finish**, espera el Gradle Sync y ejecuta **Run ▶**.

La plantilla **no** crea `NavigationRoot.kt` ni la carpeta `navigation/`: solo `MainActivity.kt`, `ui/theme/` y los tests de ejemplo. Ese archivo lo creas tú cuando llegue la navegación.

## Qué cambió con Kotlin 2.0

Hasta Kotlin 1.9, el compilador de Compose era una pieza aparte, con su propia versión atada a la de Kotlin:

```kotlin
// Kotlin 1.9: YA NO SE USA
composeOptions {
    kotlinCompilerExtensionVersion = "1.5.14"
}
```

Desde Kotlin 2.0, el Compose Compiler **viaja con Kotlin** y se activa como un plugin de Gradle con la **misma versión que Kotlin**. Ese bloque `composeOptions` se borra.

> Un proyecto nuevo de Android Studio actual ya viene así. Los pasos siguientes son para **migrar** un proyecto viejo (como el repo del curso).

### 1. Declarar el plugin en `libs.versions.toml`

```toml
[versions]
kotlin = "2.0.0"   # usa la versión que traiga tu plantilla

[plugins]
compose-compiler = { id = "org.jetbrains.kotlin.plugin.compose", version.ref = "kotlin" }
```

Ojo con las **comillas** alrededor del `id`: sin ellas el sync falla.

### 2. Aplicarlo en los dos `build.gradle.kts`

`build.gradle.kts` **raíz** (el del proyecto):

```kotlin
plugins {
    alias(libs.plugins.android.application) apply false
    alias(libs.plugins.kotlin.android) apply false
    alias(libs.plugins.compose.compiler) apply false
}
```

`app/build.gradle.kts`:

```kotlin
plugins {
    alias(libs.plugins.android.application)
    alias(libs.plugins.kotlin.android)
    alias(libs.plugins.compose.compiler)
}
```

¿Por qué `apply false` en la raíz? Porque ahí solo **declaras** el plugin y su versión una vez para todo el proyecto; quien lo **aplica** es el módulo `app`.

**Truco para distinguir los dos archivos:** presiona `Shift` dos veces en Android Studio y busca `build.gradle`. El global aparece con un punto (`.`) al lado; el del módulo, con `app`.

### 3. Alinear KSP

KSP genera código para Room y Hilt, y **tiene que ser compatible con tu versión de Kotlin**. En el curso:

```toml
[versions]
kotlin = "2.0.0"
ksp = "2.0.0-1.0.22"   # prefijo = versión de Kotlin
```

Durante años la regla fue "el prefijo de KSP es la versión de Kotlin". En versiones recientes de KSP ese esquema cambió, así que **no copies números de un curso viejo**: revisa la página de [releases de KSP](https://github.com/google/ksp/releases) para la versión que corresponde a tu Kotlin.

Después de cada cambio: **Sync Now**.

## Verificar que todo corre

1. Ten listo un emulador (**Device Manager → Create Virtual Device**) o un teléfono conectado con depuración USB.
2. **Run ▶**.
3. Si la app abre en el emulador sin errores de compilación, la configuración está completa.

## Errores típicos

| Síntoma | Causa probable |
| --- | --- |
| Android Studio dice que el proyecto "no es compatible" | La versión de AGP está fuera del rango que soporta tu Android Studio |
| `Starting in Kotlin 2.0, the Compose Compiler Gradle plugin is required` | Falta el plugin `compose-compiler` |
| Error de KSP al sincronizar | La versión de KSP no corresponde a la de Kotlin |
| Sync falla después de editar el toml | Falta una comilla o hay un alias mal escrito |

-----

## Resumen en 5 líneas

1. Si el repo del curso no abre en tu Android Studio, crea un proyecto nuevo con **Empty Activity** y copia solo el código, nunca los archivos de Gradle viejos.
2. Desde Kotlin 2.0, Compose usa el plugin `org.jetbrains.kotlin.plugin.compose` con la misma versión que Kotlin; `composeOptions` se borra.
3. El plugin se declara en el toml, se aplica con `apply false` en la raíz y sin `apply false` en `app`.
4. KSP tiene que ser compatible con Kotlin: consulta sus releases en lugar de copiar números viejos.
5. La configuración está lista cuando **Run ▶** abre la app en el emulador sin errores.

-----

## Siguiente lección

Temas, colores y tipografía con Material 3 (próximamente).
