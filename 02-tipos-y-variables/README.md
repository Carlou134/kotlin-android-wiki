# Tipos y variables

En esta carpeta aprendes a **guardar datos** en tu programa: cómo declarar variables, cuándo pueden cambiar y cuándo no, y cómo trabajar con los dos tipos de datos que más vas a usar: **texto** y **números**.

-----

## Antes de empezar

Conviene que ya hayas leído la carpeta anterior: [Fundamentos](../01-fundamentos/README.md). Vas a escribir todo dentro de `main()` y mostrar resultados con `println()`.

Si en algún momento aparece una palabra que no conoces, búscala en el [Glosario](Glosario.md).

-----

## Orden de lectura

Las lecciones se apoyan una en la otra, así que conviene leerlas en este orden:

| Lección | Qué aprendes | Qué necesitas saber antes |
| --- | --- | --- |
| [1. Tipos y variables](01-tipos-y-variables.md) | `val` y `var`, tipos básicos, inferencia de tipos y conversiones | `main()` y `println()` |
| [2. Texto: String y Char](02-texto.md) | `String` y `Char`, concatenación, plantillas, `readLine()` y secuencias de escape | `val`, `var` y tipos |
| [3. Números](03-numeros.md) | Tipos numéricos, operadores, orden de operaciones y la librería `Math` | `val`, `var` y tipos |

-----

## El mapa completo en una mirada

```
Variables  ->  "guardo un dato con nombre"
   val nombre = "Ana"      (no cambia)
   var edad = 30           (puede cambiar)
   el tipo se infiere o se declara: val activo: Boolean = true

Texto      ->  "trabajo con palabras"
   String "hola", Char 'h', plantillas "Hola, $nombre", readLine(), \n

Números    ->  "hago cálculos"
   Int, Long, Double, Float
   + - * / %   +=  ++   orden de operaciones   Math
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

Ya sabes guardar un dato. Ahora aprendes a guardar muchos datos juntos: [Colecciones](../03-colecciones/README.md).
