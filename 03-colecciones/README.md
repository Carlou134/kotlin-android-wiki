# Colecciones

En esta carpeta aprendes a **guardar muchos datos juntos** en una sola estructura. Kotlin tiene tres colecciones principales y cada una resuelve un problema distinto: **listas** (orden), **sets** (sin duplicados) y **maps** (buscar por clave).

-----

## Antes de empezar

Conviene que ya hayas leído la carpeta anterior: [Tipos y variables](../02-tipos-y-variables/README.md). Las colecciones se guardan en variables con `val` o `var`, y la diferencia entre mutable e inmutable vuelve a aparecer acá.

Si en algún momento aparece una palabra que no conoces, búscala en el [Glosario](Glosario.md).

-----

## Orden de lectura

Las lecciones se apoyan una en la otra, así que conviene leerlas en este orden:

| Lección | Qué aprendes | Qué necesitas saber antes |
| --- | --- | --- |
| [1. Listas](01-listas.md) | `listOf`, `mutableListOf`, índices, `size`, funciones de listas y arrays | `val`, `var` y tipos |
| [2. Sets](02-sets.md) | `setOf`, `mutableSetOf`, `elementAt()` y funciones de sets | Listas |
| [3. Maps](03-maps.md) | `mapOf`, `mutableMapOf`, claves, valores, `put()` y `remove()` | Listas y sets |

-----

## El mapa completo en una mirada

```
Lista  ->  "elementos en orden, con repetidos"
   listOf("a", "b", "a")        lista[0]
   mutableListOf(...)           add(), remove()

Set    ->  "elementos únicos"
   setOf("a", "b", "a") -> [a, b]
   mutableSetOf(...)            elementAt(0)

Map    ->  "busco un valor por su clave"
   mapOf("ana" to 30)           mapa["ana"]
   mutableMapOf(...)            put(), remove()

En las tres: xxxOf = solo lectura, mutableXxxOf = modificable
```

-----

## Cómo está armada cada lección

1. **En una frase:** qué es, sin jerga.
2. **Antes de empezar:** qué tienes que saber y las palabras nuevas.
3. **El contenido:** la explicación con ejemplos.
4. **Resumen en 5 líneas:** para repasar.
5. **Siguiente lección:** para seguir el orden.

-----

## Después de esta carpeta

Ya sabes guardar muchos datos. Ahora aprendes a tomar decisiones y a repetir tareas sobre ellos: [Control de flujo](../04-control-de-flujo/README.md).
