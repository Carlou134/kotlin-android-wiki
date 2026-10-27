# Jetpack Compose

En esta carpeta empieza el recorrido de Android. Aprendes qué es **Jetpack Compose**, por qué cambia la forma de pensar las pantallas (de imperativo a declarativo) y cómo dejar listo un proyecto con Kotlin 2.x y Compose sin pelearte con versiones incompatibles. El hilo conductor es una **to-do app**: lista de tareas con modo claro y oscuro, creación de tareas y listas de pendientes y completadas.

-----

## Antes de empezar

Conviene que ya hayas leído [Gradle con Kotlin DSL y version catalog](../20-build-y-devops/01-gradle-y-version-catalog.md) (el `libs.versions.toml` que vas a tocar), [Funciones de orden superior](../09-funcional/01-funciones-de-orden-superior.md) (Compose es puro lambdas) y [kotlinx.serialization](../11-serializacion-y-tiempo/01-kotlinx-serialization.md) (la navegación la usa).

También necesitas:

* Android Studio instalado.
* Un emulador configurado o un teléfono conectado.
* Manejo básico de Git y GitHub.

Si en algún momento aparece una palabra que no conoces, búscala en el [Glosario](Glosario.md).

-----

## Orden de lectura

| Lección | Qué aprendes | Qué necesitas saber antes |
| --- | --- | --- |
| [1. Qué es Jetpack Compose](01-que-es-compose.md) | Enfoque declarativo vs imperativo, por qué Compose es el estándar, qué app vas a construir | Kotlin básico |
| [2. Configuración del proyecto](02-configuracion-del-proyecto.md) | Crear el proyecto, Kotlin 2.x, el plugin Compose Compiler, KSP y verificar en el emulador | Version catalog |
| [3. Tema con Material 3](03-tema-con-material-3.md) | Tokens de color, escala tipográfica, Material Theme Builder, Google Fonts y modo claro/oscuro | Configuración del proyecto |
| [4. Composables, modifiers y layouts](04-composables-y-layouts.md) | `@Composable`, la regla del modifier, `@Preview`, estado con `remember`, `Row`/`Column`, íconos y `@PreviewParameter` | Tema con Material 3 |

-----

## El mapa completo en una mirada

```
Imperativo (XML + Views)  ->  "cómo actualizo esta vista cuando algo cambia"
   findViewById -> textView.text = ... -> boton.isEnabled = ...

Declarativo (Compose)     ->  "cómo se ve esta pantalla según el estado"
   @Composable fun Pantalla(estado) { Text(estado.titulo) }
   el estado cambia -> Compose recompone solo lo necesario

Proyecto                  ->  "versiones alineadas o nada compila"
   libs.versions.toml: kotlin = "2.x"   ->  compose-compiler usa la MISMA versión
   ksp debe ser compatible con kotlin
   build.gradle.kts raíz: apply false · app: alias(libs.plugins.compose.compiler)

Tema (Material 3)         ->  "el sistema de diseño, definido una vez"
   colorScheme: primary/onPrimary · surface/onSurface · error/onError
   typography: display > headline > title > body > label (Large/Medium/Small)
   TodoAppTheme { ... }  ->  lightScheme o darkScheme según isSystemInDarkTheme()

Composables (Lego)        ->  "piezas chicas que se apilan"
   @Composable fun Pieza(modifier: Modifier = Modifier)
   Modifier.background(...).padding(...)   ← el orden importa
   var x by remember { mutableStateOf(...) }  ->  cambia x -> recompone
   Row (horizontal) · Column (vertical) · Box (encimados)
```

-----

## Cómo está armada cada lección

1. **En una frase:** qué es, sin jerga.
2. **Antes de empezar:** qué tienes que saber y las palabras nuevas.
3. **El contenido:** la explicación con ejemplos.
4. **Resumen en 5 líneas:** para repasar.
5. **Siguiente lección:** para seguir el orden.

-----

## Recursos

* [Documentación oficial de Jetpack Compose](https://developer.android.com/develop/ui/compose/documentation)
* [Compose Compiler Gradle plugin](https://developer.android.com/develop/ui/compose/compiler)
* [Compatibilidad Compose ↔ Kotlin](https://developer.android.com/jetpack/androidx/releases/compose-kotlin)
* [Releases de KSP](https://github.com/google/ksp/releases)
* [Material 3 en Compose](https://developer.android.com/develop/ui/compose/designsystems/material3)
* [Material Theme Builder](https://material-foundation.github.io/material-theme-builder/)

-----

## Después de esta carpeta

La etapa sigue creciendo a medida que avanza el curso. Vuelve al índice: [README principal](../README.md).
