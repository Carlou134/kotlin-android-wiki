# Qué es Jetpack Compose

## En una frase

Jetpack Compose es la herramienta oficial de Google para construir interfaces en Android de forma **declarativa**: describes cómo debe verse la pantalla según el estado actual, y cuando el estado cambia, Compose vuelve a dibujar solo lo necesario.

-----

## Antes de empezar

Conviene que ya sepas:

* Funciones y lambdas, porque cada pieza de UI es una función: [Funciones de orden superior](../09-funcional/01-funciones-de-orden-superior.md).
* Clases y data classes, porque el estado de una pantalla suele ser una: [Data classes](../06-poo/03-data-classes.md).

Palabras nuevas (también están en el [Glosario](Glosario.md)):

* **Imperativo:** Estilo en el que das órdenes paso a paso para modificar la UI ("buscá esta vista, cambiale el texto").
* **Declarativo:** Estilo en el que describes el resultado según el estado ("con este estado, la pantalla se ve así").
* **Estado (state):** Los datos que determinan qué muestra la pantalla en un momento dado.
* **Composable:** Función marcada con `@Composable` que describe una parte de la UI.
* **Recomposición:** Cuando el estado cambia, Compose vuelve a ejecutar los composables afectados para actualizar la pantalla.

-----

## El problema: el enfoque imperativo

Antes de Compose, la UI de Android se escribía en XML y se manipulaba desde el código:

```kotlin
// Imperativo: TÚ eres responsable de mantener la vista sincronizada
val titulo = findViewById<TextView>(R.id.titulo)
val boton = findViewById<Button>(R.id.boton_completar)

fun alCompletarTarea(tarea: Tarea) {
    titulo.text = tarea.nombre
    titulo.paintFlags = titulo.paintFlags or Paint.STRIKE_THRU_TEXT_FLAG
    boton.isEnabled = false
}
```

Cada cambio de datos exige recordar **qué vistas tocar y cómo**. Si te olvidas de una línea, la pantalla queda mostrando un estado que ya no es real. Ese es el origen de muchos bugs: la vista y los datos se desincronizan.

## La solución: el enfoque declarativo

```kotlin
// Declarativo: describes la pantalla EN FUNCIÓN del estado
@Composable
fun TareaItem(tarea: Tarea, onCompletar: () -> Unit) {
    Row {
        Text(
            text = tarea.nombre,
            textDecoration = if (tarea.completada) TextDecoration.LineThrough else null
        )
        Button(onClick = onCompletar, enabled = !tarea.completada) {
            Text("Completar")
        }
    }
}
```

No hay `findViewById` ni asignaciones manuales. Si `tarea.completada` pasa a `true`, Compose **recompone** `TareaItem` y el tachado y el botón deshabilitado aparecen solos. La pantalla es una función del estado: `UI = f(estado)`.

El cambio de fondo no es estético, es de mentalidad:

| Imperativo | Declarativo |
| --- | --- |
| "¿Cómo actualizo esta vista cuando algo cambia?" | "¿Cómo se ve esta pantalla con este estado?" |
| Tú sincronizas vista y datos | El framework sincroniza por ti |
| XML + código Kotlin separados | Todo en Kotlin |

## Por qué importa dominarlo

* **Es el estándar oficial** de Android para interfaces nuevas.
* **Menos código y menos bugs:** no manipulas vistas a mano, así que desaparece una categoría entera de errores.
* **Arquitecturas modernas:** Compose empuja patrones como estado inmutable, flujo unidireccional de datos y separación UI/lógica (los que piden las vacantes senior).
* **No es solo Android:** con **Compose Multiplatform** el mismo modelo sirve para iOS, escritorio y web.

## La app que vas a construir

Una **to-do app** con:

* Modo claro y modo oscuro.
* Creación de nuevas tareas.
* Listas de tareas pendientes y completadas.

El curso publica un repositorio en GitHub con una **rama de inicio y otra de fin por cada clase**. Sirve para comparar el antes y el después de cada sesión con `git diff` y saltar a cualquier punto sin reproducir todo a mano. Si prefieres construir desde cero (lo recomendado para aprender de verdad), usa esas ramas solo como referencia cuando te trabes.

-----

## Resumen en 5 líneas

1. Compose es la forma oficial y declarativa de construir UI en Android: todo en Kotlin, sin XML.
2. Imperativo = tú actualizas cada vista a mano; declarativo = describes la UI según el estado.
3. Un composable es una función `@Composable`; cuando su estado cambia, Compose la recompone.
4. Menos manipulación manual de vistas significa menos código y menos bugs de desincronización.
5. El proyecto del curso es una to-do app con temas claro/oscuro, creación de tareas y listas pendientes/completadas.

-----

## Siguiente lección

[2. Configuración del proyecto](02-configuracion-del-proyecto.md): crear el proyecto y alinear Kotlin, Compose y KSP.
