# Glosario

Las palabras que aparecen en las lecciones de esta carpeta, explicadas de forma simple. Están en orden alfabético.

-----

**AGP (Android Gradle Plugin).** El plugin de Gradle que sabe compilar apps Android. Cada versión de Android Studio soporta un rango de AGP.

**`apply false`.** En el `build.gradle.kts` raíz, declara un plugin y su versión sin aplicarlo; lo aplican los módulos.

**Compose Compiler.** Plugin del compilador de Kotlin que transforma las funciones `@Composable`. Desde Kotlin 2.0 usa la misma versión que Kotlin.

**Compose Multiplatform.** Variante de Compose que permite usar el mismo modelo de UI en Android, iOS, escritorio y web.

**Composable.** Función marcada con `@Composable` que describe una parte de la UI.

**Declarativo.** Estilo en el que describes el resultado según el estado, y el framework se encarga de actualizar la pantalla.

**Emulador.** Un dispositivo Android virtual que corre en tu PC.

**Empty Activity.** La plantilla de Android Studio para un proyecto nuevo con Compose.

**Estado (state).** Los datos que determinan qué muestra la pantalla en un momento dado.

**Gradle Sync.** Cuando Android Studio lee los archivos de Gradle y arma el modelo del proyecto.

**Imperativo.** Estilo en el que das órdenes paso a paso para modificar cada vista.

**Jetpack Compose.** La herramienta oficial de Google para construir interfaces Android de forma declarativa en Kotlin.

**KSP (Kotlin Symbol Processing).** Procesador de anotaciones que genera código al compilar. Lo usan Room y Hilt; su versión debe ser compatible con la de Kotlin.

**`NavigationRoot`.** Composable que centraliza la navegación de la app: el `NavHost`, el `NavController` y las rutas. No lo crea la plantilla.

**Recomposición.** Cuando el estado cambia, Compose vuelve a ejecutar los composables afectados para actualizar la pantalla.
