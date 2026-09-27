# Control de flujo

Hasta acá tu programa se ejecutaba de arriba hacia abajo, siempre igual. En esta carpeta aprendes a **tomar decisiones** (condicionales) y a **repetir tareas** (bucles), incluso recorriendo las colecciones que ya conoces.

-----

## Antes de empezar

Conviene que ya hayas leído [Tipos y variables](../02-tipos-y-variables/README.md), porque las condiciones son valores `Boolean`, y [Colecciones](../03-colecciones/README.md), porque los bucles las recorren.

Si en algún momento aparece una palabra que no conoces, búscala en el [Glosario](Glosario.md).

-----

## Orden de lectura

Las lecciones se apoyan una en la otra, así que conviene leerlas en este orden:

| Lección | Qué aprendes | Qué necesitas saber antes |
| --- | --- | --- |
| [1. Condicionales](01-condicionales.md) | `if`/`else`, operadores de comparación y lógicos, `when` y rangos | `Boolean` y operadores numéricos |
| [2. Bucles](02-bucles.md) | `for`, `while`, `do..while`, bucles anidados, `break`, `continue` y etiquetas | Condicionales, rangos y colecciones |

-----

## El mapa completo en una mirada

```
Condicionales  ->  "¿qué camino toma el programa?"
   if / else if / else
   comparación: > < >= <= == !=     lógicos: && || !
   when (valor) { ... }             rangos: 1..10, in

Bucles         ->  "¿cuántas veces repito esto?"
   for (i in 1..5)          rangos: until, downTo, step
   for (x in lista)         for ((clave, valor) in mapa)
   while / do..while
   break, continue, etiquetas (externo@)
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

Ya sabes decidir y repetir. Ahora aprendes a empaquetar esa lógica para reutilizarla: [Funciones](../05-funciones/README.md).
