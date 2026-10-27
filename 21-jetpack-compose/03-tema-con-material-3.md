# Tema con Material 3

## En una frase

El tema de una app Compose es su **sistema de diseño**: una paleta de colores con tokens semánticos, una escala tipográfica y formas, definidos una vez con **Material 3** y aplicados a toda la UI, con un esquema para modo claro y otro para modo oscuro.

-----

## Antes de empezar

Conviene que ya sepas:

* Qué es un composable y cómo envuelve contenido: [Qué es Jetpack Compose](01-que-es-compose.md).
* Cómo agregar una librería al version catalog: [Configuración del proyecto](02-configuracion-del-proyecto.md).
* Lambdas como parámetro (`content: @Composable () -> Unit`): [Funciones de orden superior](../09-funcional/01-funciones-de-orden-superior.md).

Palabras nuevas (también están en el [Glosario](Glosario.md)):

* **Material 3:** El sistema de diseño de Google y el que usan por defecto los componentes de Compose.
* **Token de color:** Un nombre con intención (`primary`, `surface`, `error`) en lugar de un color fijo.
* **Color scheme:** El conjunto de todos los tokens de color para un modo (claro u oscuro).
* **Material Theme Builder:** Herramienta web que genera el tema completo a partir de un color base.
* **Downloadable fonts:** Fuentes que se descargan en tiempo de ejecución desde Google Fonts en lugar de venir dentro de la app.

-----

## Material 3 vs Material 2

Material 3 es el estándar actual. Hay dos librerías con nombres parecidos y **no se mezclan**:

| Librería | Import | Estado |
| --- | --- | --- |
| Material 2 | `androidx.compose.material.*` | Legacy |
| **Material 3** | `androidx.compose.material3.*` | **La actual** |

La plantilla de Android Studio ya trae `androidx.compose.material3:material3`, con la versión controlada por el Compose BOM. Cuando el IDE te sugiera un import, verifica que diga **`material3`**: un `Button` de Material 2 dentro de un tema de Material 3 no toma tus colores.

## Colores: tokens con intención

Material 3 no te pide "azul" o "gris": te pide **roles**. Cada color de fondo tiene un par `on...` para lo que va encima (texto, íconos), así el contraste está garantizado.

| Token | Para qué |
| --- | --- |
| `primary` / `onPrimary` | El color de la marca y lo que va sobre él (botón principal y su texto) |
| `secondary` / `onSecondary` | Elementos secundarios, refuerzan jerarquía |
| `tertiary` / `onTertiary` | Acentos de contraste |
| `primaryContainer` / `onPrimaryContainer` | Elementos interactivos que destacan menos que `primary` (chips, tarjetas) |
| `error` / `onError`, `errorContainer` / `onErrorContainer` | Estados de error |
| `surface` / `onSurface` | Fondos neutros y su contenido |
| `surfaceDim` / `surfaceBright` | Variantes de `surface` más oscura o más clara, para crear jerarquía sin usar color de marca |

En el código se leen desde el tema, **nunca con colores fijos**:

```kotlin
Text(
    text = "Comprar pan",
    color = MaterialTheme.colorScheme.onSurface
)
```

Si escribes `Color(0xFF000000)` a mano, en modo oscuro tendrás texto negro sobre fondo negro. Con el token, Compose elige el color correcto para cada modo.

## Tipografía: 5 familias × 3 tamaños

De mayor a menor, cada una con `Large`, `Medium` y `Small`:

| Familia | Para qué | Ejemplo |
| --- | --- | --- |
| `display` | Textos enormes, impacto visual | Un número grande en un contador |
| `headline` | Encabezados destacados | Título de pantalla |
| `title` | Títulos de secciones | "Pendientes", "Completadas" |
| `body` | Párrafos y contenido | La descripción de una tarea |
| `label` | Etiquetas, chips, menús | El texto de un botón |

```kotlin
Text("Pendientes", style = MaterialTheme.typography.titleMedium)
```

Eliges la **intención** y el tamaño viene del tema: si mañana cambias la fuente, cambia en toda la app.

## Generar el tema con Material Theme Builder

1. Entra a [Material Theme Builder](https://material-foundation.github.io/material-theme-builder/).
2. Elige un **source color**. En la to-do app del curso se eligió un color que recuerda a una hoja de papel, para reforzar la idea del producto.
3. Elige la tipografía. El curso cambia Roboto (la fuente por defecto de Android) por **Poppins**, de Google Fonts.
4. **Export → Jetpack Compose (Theme.kt)**. Descarga un paquete con:
   * `ui/theme/`: `Color.kt`, `Theme.kt`, `Type.kt`
   * `res/values/`: el archivo de certificados de fuentes

## Integrarlo al proyecto

### 1. Reemplazar los archivos del tema

Copia `Color.kt`, `Theme.kt` y `Type.kt` sobre los de `ui/theme/` y **corrige la línea `package`** de cada uno para que coincida con tu proyecto:

```kotlin
package com.example.todo_app.ui.theme
```

### 2. Copiar el certificado de fuentes

Copia el archivo de certificados a `app/src/main/res/values/` (en el proyecto queda como `font_certs.xml`). Define el array `com_google_android_gms_fonts_certs`.

¿Por qué hace falta? Con downloadable fonts, la fuente **no viene dentro de tu app**: la descarga Google Play Services en tiempo de ejecución. El certificado le permite a Android verificar que el proveedor es realmente Google antes de cargar la fuente. Sin él, la fuente no se renderiza.

Si Theme Builder no te da el archivo, está en el [repo oficial de ejemplos de Google](https://github.com/googlearchive/android-DownloadableFonts/blob/master/app/src/main/res/values/font_certs.xml).

### 3. Agregar la librería de Google Fonts

`Type.kt` queda en rojo porque `GoogleFont` está en una librería aparte. Con el cursor sobre el rojo, **Alt+Enter** (en Mac: Option+Enter) → *Add library dependency and import*. O a mano:

```toml
# libs.versions.toml → [libraries]
androidx-compose-ui-text-google-fonts = { group = "androidx.compose.ui", name = "ui-text-google-fonts" }
```

```kotlin
// app/build.gradle.kts
implementation(libs.androidx.compose.ui.text.google.fonts)
```

Sin `version.ref`: la versión la pone el Compose BOM.

### Cómo queda `Type.kt`

```kotlin
package com.example.todo_app.ui.theme

import androidx.compose.ui.text.font.FontFamily
import androidx.compose.ui.text.googlefonts.Font          // ← googlefonts, NO text.font.Font
import androidx.compose.ui.text.googlefonts.GoogleFont
import com.example.todo_app.R                              // ← el R de TU app

val provider = GoogleFont.Provider(
    providerAuthority = "com.google.android.gms.fonts",
    providerPackage = "com.google.android.gms",
    certificates = R.array.com_google_android_gms_fonts_certs
)

val bodyFontFamily = FontFamily(
    Font(googleFont = GoogleFont("Poppins"), fontProvider = provider)
)
```

## Simplificar `Theme.kt`

El archivo generado trae esquemas de contraste medio y alto (`mediumContrastLightColorScheme`, etc.) que una app sencilla no usa. Deja solo `lightScheme` y `darkScheme`.

La función generada se llama `AppTheme` y recibe:

* `darkTheme`: si el sistema está en modo oscuro, vía `isSystemInDarkTheme()`.
* `dynamicColor`: en Android 12+ adapta los colores al fondo de pantalla del usuario.
* `content`: lo que el tema envuelve.

El curso quita `dynamicColor` para que la app mantenga **sus** colores sin importar el wallpaper:

```kotlin
@Composable
fun TodoAppTheme(
    darkTheme: Boolean = isSystemInDarkTheme(),
    content: @Composable () -> Unit
) {
    val colorScheme = if (darkTheme) darkScheme else lightScheme

    MaterialTheme(
        colorScheme = colorScheme,
        typography = AppTypography,
        content = content
    )
}
```

Para renombrar `AppTheme` usa **clic derecho → Refactor → Rename** (o **Shift+F6**), **no** edites a mano: el IDE cambia el nombre en todos los archivos que lo usan a la vez (`MainActivity`, el `@Preview`).

## Probar claro y oscuro

1. Envuelve el contenido de `MainActivity` con tu tema:
   ```kotlin
   setContent {
       TodoAppTheme {
           // pantallas
       }
   }
   ```
2. **Run ▶**.
3. En el emulador, abre el panel rápido (desliza desde arriba), entra en modo edición (✏️) y arrastra el mosaico **Dark theme** al panel. Al activarlo, la misma pantalla cambia de fondo claro a oscuro con los tokens de tu paleta.

También puedes ver ambos modos sin el emulador, en el preview:

```kotlin
@Preview(uiMode = Configuration.UI_MODE_NIGHT_YES, showBackground = true)
```

## Errores reales al integrar el tema

| Síntoma | Causa | Solución |
| --- | --- | --- |
| `GoogleFont` en rojo | Falta la librería `ui-text-google-fonts` | Agregarla al toml y al `build.gradle.kts` (Alt+Enter) |
| `R` en rojo dentro de `Type.kt` | `R` vive en el paquete raíz (`com.example.todo_app`), no en `ui.theme` | `import com.example.todo_app.R` (Alt+Enter sobre la `R`) |
| `com_google_android_gms_fonts_certs` en rojo | No existe `font_certs.xml` o `R` no se regeneró | Copiar el archivo y compilar (Ctrl+F9 o `.\gradlew assembleDebug`) |
| **Compila**, pero el editor sigue en rojo | El índice del IDE está desactualizado; el código está bien | Sync de Gradle; si no, **File → Invalidate Caches → Invalidate and Restart** |
| `Unresolved reference 'TodoappTheme'` + `@Composable invocations can only happen...` | El `Theme.kt` nuevo llama a la función `AppTheme`, pero `MainActivity` sigue usando el nombre viejo | Usar el mismo nombre en ambos lados (renombrar con Shift+F6). El segundo error es consecuencia del primero |
| El parámetro `googleFont` da error | Se importó `androidx.compose.ui.text.font.Font` | Importar `androidx.compose.ui.text.googlefonts.Font` |
| La fuente no cambia en el emulador | El emulador no tiene Google Play Services o no tiene internet | Usar una imagen de emulador con Google Play |

**Regla general:** cuando aparecen varios errores juntos, lee **el primero**. Un `Unresolved reference` suele arrastrar a todos los demás.

-----

## Resumen en 5 líneas

1. Material 3 (`androidx.compose.material3`) es el sistema de diseño actual: colores, tipografía y formas definidos una vez en el tema.
2. Los colores son tokens con intención (`primary`/`onPrimary`, `surface`/`onSurface`); en el código se leen con `MaterialTheme.colorScheme`, nunca con colores fijos.
3. La tipografía tiene 5 familias (display, headline, title, body, label) × 3 tamaños, y se lee con `MaterialTheme.typography`.
4. Material Theme Builder genera `Color.kt`, `Theme.kt`, `Type.kt` y el certificado de fuentes; hay que ajustar el `package`, agregar `ui-text-google-fonts` e importar el `R` de tu app.
5. Simplifica `Theme.kt` a `lightScheme`/`darkScheme`, quita `dynamicColor` si quieres colores fijos, y renombra siempre con Refactor → Rename.

-----

## Siguiente lección

Próximamente.
