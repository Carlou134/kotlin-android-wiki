# Composables, modifiers y layouts

## En una frase

Una pantalla en Compose se arma como con piezas de Lego: **composables** pequeños (un texto, un ícono), ajustados con **modifiers** y apilados con **`Row`** y **`Column`**; si la pieza cambia con el tiempo, guarda su dato en un **estado** con `remember { mutableStateOf(...) }`.

-----

## Antes de empezar

Conviene que ya sepas:

* Qué es el enfoque declarativo y la recomposición: [Qué es Jetpack Compose](01-que-es-compose.md).
* Cómo leer colores y tipografía del tema: [Tema con Material 3](03-tema-con-material-3.md).
* Parámetros con valor por defecto y lambdas: [Funciones](../05-funciones/01-funciones.md) y [Funciones de orden superior](../09-funcional/01-funciones-de-orden-superior.md).

Palabras nuevas (también están en el [Glosario](Glosario.md)):

* **Modifier:** Objeto encadenable que ajusta tamaño, espaciado, fondo, bordes y comportamiento (clic) de un composable.
* **`@Preview`:** Anotación que dibuja un composable en el panel del IDE sin correr la app.
* **`remember`:** Guarda un valor entre recomposiciones.
* **`mutableStateOf`:** Crea un estado observable: cuando cambia, Compose recompone a quien lo lee.
* **Delegación de propiedades (`by`):** Sintaxis de Kotlin que redirige las lecturas y escrituras de una variable a otro objeto.
* **`dp` / `sp`:** Unidades independientes de la densidad de pantalla; `sp` además respeta el tamaño de letra que eligió el usuario.

-----

## El primer composable

Una función con `@Composable` describe un trozo de UI. En el curso se crea en `components/Playground.kt`:

```kotlin
@Composable
fun HelloWorld(modifier: Modifier = Modifier) {
    Text(text = "Hello, world!", modifier = modifier)
}
```

Convenciones que Compose espera:

* El nombre va en **PascalCase** (`HelloWorld`, no `helloWorld`): es un "componente", no una acción.
* No devuelve nada (`Unit`): **emite** UI, no la retorna.

## Modifier: la regla de oro

**Todo composable propio recibe `modifier: Modifier = Modifier` y lo aplica a su elemento raíz.**

```kotlin
@Composable
fun Etiqueta(texto: String, modifier: Modifier = Modifier) {
    Text(text = texto, modifier = modifier.padding(8.dp))
}

// El padre decide desde afuera, sin tocar Etiqueta:
Etiqueta("Urgente", modifier = Modifier.background(Color.Red))
```

¿Por qué? Porque el **padre** es quien sabe dónde va la pieza (márgenes, tamaño, alineación). Si `Etiqueta` no expone su modifier, nadie puede ubicarla sin modificar su código.

* Es el **primer parámetro opcional**: después de los obligatorios, antes de los demás opcionales.
* Por defecto `Modifier` (vacío), así no es obligatorio pasarlo.
* Se aplica **al raíz**, no a un hijo interno.

### El orden importa

Los modifiers se aplican **de afuera hacia adentro, en el orden en que los escribes**:

```kotlin
// 1. padding y después background: el color cubre solo el área interna
Text("Hola", modifier = Modifier.padding(16.dp).background(Color.Yellow))

// 2. background y después padding: el color cubre todo, el texto queda separado del borde
Text("Hola", modifier = Modifier.background(Color.Yellow).padding(16.dp))
```

Lo mismo pasa con `clickable`: si va **antes** de `padding`, el área tocable incluye el padding; si va **después**, solo el contenido.

## Previsualizar sin correr la app

```kotlin
@Preview(showBackground = true)
@Composable
private fun HelloWorldPreview() {
    TodoAppTheme {
        HelloWorld()
    }
}
```

* Envuélvelo en **tu tema**, si no el preview usa colores por defecto y no ves tu paleta.
* `showBackground = true` pinta un fondo para que se vea el contenido.
* Arriba a la derecha del editor: **Split** muestra código y preview juntos.
* **Start Interactive Mode** (en el panel de preview) permite hacer clic y escribir sin emulador.
* Hazlo `private`: es solo para el IDE, no para el resto de la app.

## Estado: por qué un `String` normal no alcanza

```kotlin
// ❌ No funciona: Compose no se entera del cambio
@Composable
fun ClickMe() {
    var texto = "Click me!"
    Text(texto, modifier = Modifier.clickable { texto = "Clicked!" })
}
```

Compose solo recompone cuando cambia un **estado observable**. Y además, cada recomposición vuelve a ejecutar la función desde cero, así que una variable local se reinicia.

```kotlin
// ✅ Funciona
@Composable
fun ClickMe() {
    var texto by remember { mutableStateOf("Click me!") }
    Text(texto, modifier = Modifier.clickable { texto = "Clicked!" })
}
```

Cada pieza hace una cosa:

| Pieza | Qué hace | Si falta |
| --- | --- | --- |
| `mutableStateOf` | Hace el valor **observable**: al cambiar, dispara la recomposición | La pantalla nunca cambia |
| `remember` | **Guarda** el estado entre recomposiciones | El estado se crea de nuevo en cada recomposición y vuelve a `"Click me!"` |
| `by` | Permite usar `texto` en vez de `texto.value` | Hay que escribir `.value` en todos lados |

### El `by` en rojo

`by` es **delegación de propiedades**: Kotlin traduce leer `texto` a `getValue(...)` y asignarlo a `setValue(...)`. Esas dos funciones son **extension functions** de `androidx.compose.runtime`, y una extension function no existe para el compilador si no la importas:

```kotlin
import androidx.compose.runtime.getValue   // necesaria para val y var
import androidx.compose.runtime.setValue   // necesaria solo para var
```

**Alt+Enter** sobre el `by` las importa (a veces una sola; repite si sigue en rojo).

### `remember` no sobrevive a todo

Si rotas el teléfono, la Activity se recrea y `remember` pierde el valor. Para eso existen `rememberSaveable` o el `ViewModel` (más adelante en el curso).

## Layouts: `Row` y `Column`

* **`Row`:** hijos en horizontal.
* **`Column`:** hijos en vertical.
* (**`Box`:** hijos apilados uno encima de otro.)

```kotlin
@Composable
fun TareaFila(nombre: String, modifier: Modifier = Modifier) {
    Row(
        modifier = modifier
            .fillMaxWidth()
            .padding(16.dp),
        verticalAlignment = Alignment.CenterVertically,
        horizontalArrangement = Arrangement.spacedBy(8.dp)
    ) {
        Icon(
            imageVector = Icons.Default.Favorite,
            contentDescription = "Favorita",
            modifier = Modifier
                .border(1.dp, MaterialTheme.colorScheme.outline, RoundedCornerShape(50))
                .padding(4.dp)
        )
        Text(nombre, style = MaterialTheme.typography.bodyLarge)
        Spacer(modifier = Modifier.weight(1f))   // empuja lo siguiente al extremo
        Text("Hoy", style = MaterialTheme.typography.labelSmall)
    }
}
```

Las propiedades tienen un **eje principal** (la dirección del layout) y un **eje cruzado**:

| | Eje principal (reparto) | Eje cruzado (alineación) |
| --- | --- | --- |
| `Row` | `horizontalArrangement` | `verticalAlignment` |
| `Column` | `verticalArrangement` | `horizontalAlignment` |

* `Arrangement.spacedBy(8.dp)`: separación fija entre hijos.
* `Arrangement.SpaceBetween`: el primero a un extremo, el último al otro.
* `Modifier.weight(1f)`: el hijo ocupa todo el espacio sobrante. Con un `Spacer` sirve para empujar elementos a los extremos.

### Íconos

`Icon` recibe un `ImageVector` y **siempre** un `contentDescription` (lo lee el lector de pantalla). Si el ícono es solo decorativo, pasa `null`.

`Icons.Default.Favorite` está en la librería **`material-icons-core`**, que en las versiones recientes **ya no viene incluida** con `material3`:

```toml
# libs.versions.toml → [libraries]
androidx-compose-material-icons-core = { group = "androidx.compose.material", name = "material-icons-core" }
```

```kotlin
// app/build.gradle.kts
implementation(libs.androidx.compose.material.icons.core)
```

Si el sync pide versión porque el BOM no la gestiona, usa `version = "1.7.8"` (la última publicada). Los imports son `androidx.compose.material.icons.Icons` y `androidx.compose.material.icons.filled.Favorite` (paquete `material`, **sin el 3**, y está bien).

`core` trae unos 50 íconos básicos. Para otros, **no** uses `material-icons-extended` (es enorme y Google ya no la recomienda): descarga el ícono de [Material Symbols](https://fonts.google.com/icons) como Vector Drawable y úsalo con `painterResource(R.drawable.ic_...)`.

### `dp` y `sp`

* **`dp`** para dimensiones (padding, tamaños, bordes): se ve igual en pantallas de distinta densidad.
* **`sp`** para **texto**: además escala con el tamaño de letra que eligió el usuario en Ajustes. Usar `dp` en texto rompe la accesibilidad.

## Un preview por cada variante: `@PreviewParameter`

En vez de escribir un preview por cada dato, un **provider** le pasa varios valores al mismo preview:

```kotlin
data class IconContainer(val icon: ImageVector)

class IconProvider : PreviewParameterProvider<IconContainer> {
    override val values = sequenceOf(
        IconContainer(Icons.Default.MoreVert),
        IconContainer(Icons.Default.Star),
        IconContainer(Icons.Default.Search)
    )
}

@Preview(showBackground = true)
@Composable
private fun IconPreview(
    @PreviewParameter(IconProvider::class) container: IconContainer
) {
    TodoAppTheme {
        Icon(imageVector = container.icon, contentDescription = null)
    }
}
```

Compose dibuja **un preview por cada valor** de `values`. Combinado con `uiMode = Configuration.UI_MODE_NIGHT_YES`, validas modo claro y oscuro sin emulador.

## Si vienes de React

El modelo mental es casi el mismo (`UI = f(estado)`):

| React | Compose |
| --- | --- |
| Componente funcional | Función `@Composable` |
| Props | Parámetros |
| `useState` | `remember { mutableStateOf(...) }` |
| Re-render | Recomposición |
| *Lifting state up* | *State hoisting* |
| `useEffect` | `LaunchedEffect` / `DisposableEffect` |
| `useMemo` | `remember(key) { ... }` |
| Context | `CompositionLocal` (así funciona `MaterialTheme`) |
| `className` / `style` | `Modifier` |
| `display: flex` | `Row` / `Column` |

Diferencias que importan:

1. **No hay virtual DOM:** Compose registra qué composable leyó qué estado y recompone solo esos; los demás se saltean.
2. **Se actualiza asignando** (`texto = "Hola"`), no con un setter. Pero si guardas una lista mutable y haces `lista.add(...)`, **Compose no se entera**: reemplázala (`lista = lista + nuevo`) o usa `mutableStateListOf`.
3. **`remember` + `mutableStateOf` son dos cosas separadas.** En React, `useState` hace ambas; en Compose puedes olvidarte de una.
4. **La rotación recrea la pantalla:** `remember` no sobrevive; `rememberSaveable` o `ViewModel` sí.

## Errores reales

| Síntoma | Causa | Solución |
| --- | --- | --- |
| `by` en rojo en `var x by remember { ... }` | Faltan `getValue` / `setValue` de `androidx.compose.runtime` | Alt+Enter sobre el `by` |
| `Icons.Default.Favorite` no aparece | `material-icons-core` ya no viene con `material3` | Agregar la dependencia |
| El texto no cambia al hacer clic | Variable normal en vez de `mutableStateOf` | Usar estado observable |
| El estado vuelve al valor inicial solo | `mutableStateOf` sin `remember` | Envolver en `remember { }` |
| El fondo no cubre lo que esperabas | Orden de los modifiers | `background` antes o después de `padding` según lo que quieras |
| El preview se ve con colores raros | No está envuelto en el tema | `TodoAppTheme { ... }` dentro del preview |

-----

## Resumen en 5 líneas

1. Un composable es una función `@Composable` en PascalCase que emite UI; todo composable propio recibe `modifier: Modifier = Modifier` y lo aplica al raíz.
2. Los modifiers se aplican en orden: `padding` antes o después de `background` da resultados distintos.
3. El estado que cambia va en `var x by remember { mutableStateOf(...) }`: `mutableStateOf` lo hace observable, `remember` lo conserva, `by` necesita `getValue`/`setValue`.
4. `Row` (horizontal) y `Column` (vertical) reparten en el eje principal con `Arrangement` y alinean en el cruzado con `Alignment`; `weight(1f)` ocupa el espacio sobrante.
5. `@Preview` (con el tema y `showBackground`), el modo interactivo y `@PreviewParameter` permiten validar la UI sin emulador.

-----

## Siguiente lección

[5. Fases de Compose y recomposición](05-fases-y-recomposicion.md): cómo pinta Compose y cómo evitar recomponer de más.
