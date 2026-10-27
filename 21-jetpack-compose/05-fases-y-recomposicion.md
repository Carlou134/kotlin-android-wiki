# Fases de Compose y recomposición

## En una frase

Compose pinta cada frame en tres fases (**composición → layout → dibujo**), y cuando cambia un estado vuelve a ejecutar los composables que **leen** ese estado; si diseñas mal quién lee qué, recompones de más y la pantalla se vuelve lenta.

-----

## Antes de empezar

Conviene que ya sepas:

* Qué es un estado y cómo se crea con `remember { mutableStateOf(...) }`: [Composables, modifiers y layouts](04-composables-y-layouts.md).
* Lambdas como parámetro: [Callbacks](../09-funcional/04-callbacks.md).

Palabras nuevas (también están en el [Glosario](Glosario.md)):

* **Composición:** Fase en la que Compose ejecuta tus composables y arma el árbol de UI.
* **Layout:** Fase en la que se mide cada nodo y se le asigna una posición (x, y).
* **Dibujo (drawing):** Fase en la que los nodos se pintan en pantalla.
* **Recomposición saltada (skipped):** Cuando Compose decide **no** volver a ejecutar un composable porque sus parámetros no cambiaron.
* **Layout Inspector:** Herramienta de Android Studio que muestra el árbol de UI y cuántas veces se recompone cada composable.
* **State hoisting:** Subir el estado al padre y pasarle a los hijos solo el valor que leen y lambdas para modificarlo.

-----

## Las tres fases

```
estado ──► 1. COMPOSICIÓN ──► 2. LAYOUT ──► 3. DIBUJO ──► píxeles
           "¿QUÉ se muestra?"  "¿DÓNDE y      "¿CÓMO se
                                de qué tamaño?" pinta?"
```

### 1. Composición: qué se muestra

Compose ejecuta tus funciones `@Composable` y arma un **árbol de nodos** con la jerarquía de la UI. Para algo como:

```kotlin
Row {
    Image(...)
    Column {
        Text("Comprar pan")
        Text("Hoy")
    }
}
```

el árbol es `Row → [Image, Column → [Text, Text]]`.

**Aquí ocurre la recomposición:** cuando un estado cambia, Compose vuelve a ejecutar esta fase para los composables afectados.

### 2. Layout: dónde y de qué tamaño

Cada nodo sigue el mismo algoritmo, recorriendo el árbol:

1. **Mide a sus hijos** (con las restricciones que le dio su padre).
2. **Decide su propio tamaño** en base a los de sus hijos.
3. **Ubica a sus hijos** en coordenadas x, y.

Aquí actúan los modifiers de tamaño y posición (`padding`, `fillMaxWidth`, `size`, `offset`). Compose mide cada hijo **una sola vez** por pasada, y por eso el layout es rápido aunque anides muchos `Row` y `Column`. Con XML, anidar layouts era caro.

### 3. Dibujo: cómo se pinta

Los nodos, ya medidos y ubicados, se pintan píxel a píxel: colores, texto, bordes, imágenes.

### Por qué importa conocerlas

Compose solo vuelve a ejecutar **las fases que hacen falta**. Si un cambio solo afecta el dibujo (un color animado), puede saltarse la composición y el layout. Cuanto más tarde en el pipeline leas un estado, menos trabajo hace Compose:

```kotlin
// Lee el estado en COMPOSICIÓN: cada cambio recompone
Box(Modifier.offset(x = desplazamiento.dp))

// Lee el estado en LAYOUT (versión lambda): no recompone, solo reubica
Box(Modifier.offset { IntOffset(desplazamiento, 0) })
```

Esto es optimización fina: úsalo cuando el valor cambie en cada frame (animaciones, scroll), no por defecto.

## La regla que explica todo: se recompone quien LEE

Compose no recompone "donde está declarado el estado", sino **los composables que leen su valor**. Leer significa usar `isShown` (no la lambda que lo modifica).

### ❌ Mal: el estado llega a quien no lo necesita

```kotlin
@Composable
fun Pantalla(modifier: Modifier = Modifier) {
    var isShown by remember { mutableStateOf(false) }

    Column(modifier) {
        HelloWorldText(isShown = isShown, onClick = { isShown = !isShown })
        MessageText(isShown = isShown)
    }
}

@Composable
fun HelloWorldText(isShown: Boolean, onClick: () -> Unit) {
    // recibe isShown aunque solo lo necesita para... nada
    Text("Hello world", modifier = Modifier.clickable { onClick() })
}
```

Cada clic cambia `isShown`. Como `HelloWorldText` **recibe** `isShown` como parámetro y su valor cambió, Compose lo recompone. Recompone una pieza que se ve exactamente igual que antes.

### ✅ Bien: cada uno recibe solo lo que usa

```kotlin
@Composable
fun Pantalla(modifier: Modifier = Modifier) {
    var isShown by remember { mutableStateOf(false) }

    Column(modifier) {
        HelloWorldText(onClick = { isShown = !isShown })   // solo la lambda
        MessageText(isShown = isShown)                     // solo el valor
    }
}

@Composable
fun HelloWorldText(onClick: () -> Unit, modifier: Modifier = Modifier) {
    Text("Hello world", modifier = modifier.clickable { onClick() })
}

@Composable
fun MessageText(isShown: Boolean, modifier: Modifier = Modifier) {
    if (isShown) Text("¡Hola!", modifier = modifier)
}
```

Ahora, en cada clic:

* `Pantalla` se recompone (lee `isShown` para pasárselo a `MessageText`), y está bien que lo haga.
* `HelloWorldText` **se salta**: su único parámetro es la lambda, que es la misma instancia de antes.
* `MessageText` se recompone, porque depende del valor. Es lo esperado.

Este patrón tiene nombre: **state hoisting**. El estado vive en el padre, que lo baja como **valor** a quien lo muestra y como **lambda** (evento) a quien lo modifica. Es el mismo *lifting state up* de React, con un flujo unidireccional: **el estado baja, los eventos suben**.

### ¿Por qué se puede saltar `HelloWorldText`?

Compose salta un composable cuando **todos sus parámetros son iguales** a los de la vez anterior. Para eso tiene que poder compararlos, y ahí entra la **estabilidad**:

* Tipos primitivos, `String`, funciones (lambdas) y `data class` con propiedades `val` estables → **estables**, Compose los compara con `equals`.
* Clases con `var` o colecciones como `List` (que podrían ser mutables por dentro) → **inestables**.

Desde Kotlin 2.0.20, el compilador de Compose activa por defecto el **strong skipping mode**, que relaja estas reglas. Con él, Compose también puede saltar composables con parámetros inestables, comparándolos por instancia, y además hace `remember` automático de las lambdas. Tu proyecto (Kotlin 2.2) ya lo tiene. Aun así, **pasar solo lo que cada composable usa** sigue siendo la regla: ningún compilador te salva de mandarle un valor que cambia a quien no lo necesita.

### Si vienes de React

En React, cuando un componente se re-renderiza, **todos sus hijos** se re-renderizan por defecto, salvo que los envuelvas en `React.memo`. En Compose es al revés: **saltar es el comportamiento por defecto** cuando los parámetros no cambian. Es como si todo tuviera `React.memo` puesto, y por eso pasar parámetros de más rompe la optimización.

## Medir con el Layout Inspector

No optimices a ciegas: **mide**.

1. Corre la app en el emulador (en modo debug, el normal de **Run ▶**).
2. Abre **Tools → Layout Inspector** (o el ícono en la barra lateral, según tu versión de Android Studio).
3. Activa **Show Recomposition Counts** (en las opciones del panel del árbol).
4. Interactúa con la app y mira dos columnas por composable:
   * **Recompositions:** cuántas veces se volvió a ejecutar.
   * **Skips:** cuántas veces Compose lo salteó.

En la versión ❌, cada clic sube el contador de `HelloWorldText`: 4, 5, 6, 7... En la versión ✅, ese contador se queda quieto y suben sus **skips**; solo `MessageText` sigue recomponiendo.

En una pantalla chica la diferencia no se nota. En una lista con cientos de items que se recomponen de más, sí: la app se traba al hacer scroll.

-----

## Resumen en 5 líneas

1. Compose pinta en tres fases: composición (qué), layout (dónde y de qué tamaño) y dibujo (cómo); la recomposición vuelve a ejecutar la composición.
2. En el layout, cada nodo mide a sus hijos una sola vez, decide su tamaño y los ubica; por eso anidar `Row`/`Column` es barato.
3. Se recompone **quien lee** el estado: pasa el valor solo a quien lo muestra y una lambda a quien lo modifica (state hoisting).
4. Compose salta un composable si sus parámetros no cambiaron; el strong skipping mode (por defecto desde Kotlin 2.0.20) amplía qué se puede saltar.
5. Mide con el Layout Inspector y "Show Recomposition Counts" antes de optimizar.

-----

## Siguiente lección

Próximamente.
