# Funciones

En esta carpeta aprendes a **empaquetar lógica** en bloques con nombre que puedes reutilizar: cómo recibir datos, devolver resultados, escribir funciones cortas y lambdas, y cómo una función puede llamarse a sí misma (recursión).

-----

## Antes de empezar

Conviene que ya hayas leído [Control de flujo](../04-control-de-flujo/README.md). Las funciones encapsulan justamente esa lógica: condicionales, bucles y colecciones.

Si en algún momento aparece una palabra que no conoces, búscala en el [Glosario](Glosario.md).

-----

## Orden de lectura

Las lecciones se apoyan una en la otra, así que conviene leerlas en este orden:

| Lección | Qué aprendes | Qué necesitas saber antes |
| --- | --- | --- |
| [1. Funciones](01-funciones.md) | Declarar y llamar funciones, argumentos, `return`, alcance, funciones de una expresión y lambdas | Control de flujo y colecciones |
| [2. Recursión](02-recursion.md) | Caso base, llamada recursiva y recursión de cola con `tailrec` | Funciones y `while` |

-----

## El mapa completo en una mirada

```
Funciones  ->  "empaqueto lógica para reutilizarla"
   fun sumar(a: Int, b: Int): Int { return a + b }
   argumentos nombrados y por defecto: saludar(name = "Ana")
   una sola expresión: fun cuadrado(n: Int) = n * n
   lambda: val dividir = { a: Int, b: Int -> a / b }

Recursión  ->  "una función que se llama a sí misma"
   caso base (cuándo parar) + llamada recursiva (con otros argumentos)
   tailrec fun ... -> el compilador la convierte en bucle
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

Ya sabes empaquetar lógica. Ahora aprendes a empaquetar datos y comportamiento juntos: [Programación orientada a objetos](../06-poo/README.md).
