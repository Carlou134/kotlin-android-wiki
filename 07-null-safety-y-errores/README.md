# Null safety y errores

En esta carpeta aprendes a escribir código que **no se cae**: cómo Kotlin te obliga a manejar los datos que pueden faltar (`null`) y cómo responder a los errores que ocurren mientras el programa se ejecuta (excepciones).

-----

## Antes de empezar

Conviene que ya hayas leído [Programación orientada a objetos](../06-poo/README.md), porque las excepciones propias son clases. También usas `readLine()`, que viste en [Texto](../02-tipos-y-variables/02-texto.md).

Si en algún momento aparece una palabra que no conoces, búscala en el [Glosario](Glosario.md).

-----

## Orden de lectura

Las lecciones se apoyan una en la otra, así que conviene leerlas en este orden:

| Lección | Qué aprendes | Qué necesitas saber antes |
| --- | --- | --- |
| [1. Nulabilidad](01-nulabilidad.md) | Tipos nulables, `?.`, `?:`, `!!` y leer datos del usuario con `readLine()` de forma segura | Tipos, texto y funciones |
| [2. Excepciones](02-excepciones.md) | `try`/`catch`/`finally`, `try` como expresión, `throw` y excepciones personalizadas | Nulabilidad y clases |

-----

## El mapa completo en una mirada

```
Nulabilidad  ->  "¿qué hago si el dato no está?"
   String (nunca null)    String? (puede ser null)
   email?.length          -> llamada segura: null si email es null
   email ?: "sin email"   -> Elvis: valor por defecto
   email!!                -> "confía en mí" (lanza excepción si es null)

Excepciones  ->  "¿qué hago si algo falla al ejecutar?"
   try { ... } catch (e: Exception) { ... } finally { ... }
   val r = try { ... } catch (...) { null }   (try es expresión)
   throw IllegalArgumentException("...")
   class MiError(msg: String) : Exception(msg)
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

Con esta carpeta terminas los fundamentos de sintaxis de Kotlin. Lo próximo que aprendas va en una nueva carpeta numerada `08-...`. Vuelve al índice: [README principal](../README.md).
