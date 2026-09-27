# Fundamentos

En esta carpeta das tus **primeros pasos** con Kotlin: qué es el lenguaje, cómo se ve un programa mínimo, qué pasa entre que presionas *Run* y ves la salida, y cómo preparar IntelliJ IDEA para trabajar cómodo.

-----

## Antes de empezar

Es el punto de partida del wiki: no necesitas saber nada antes.

Si en algún momento aparece una palabra que no conoces, búscala en el [Glosario](Glosario.md).

-----

## Orden de lectura

Las lecciones se apoyan una en la otra, así que conviene leerlas en este orden:

| Lección | Qué aprendes | Qué necesitas saber antes |
| --- | --- | --- |
| [1. Introducción a Kotlin](01-introduccion.md) | Qué es Kotlin, `main()`, `println`/`print`, comentarios y errores | Nada |
| [2. Del editor a la salida](02-del-editor-a-la-salida.md) | Compilador, bytecode y JVM; compilar y ejecutar desde la terminal | `main()` y `println()` |
| [3. Entorno IntelliJ IDEA](03-entorno-intellij.md) | Instalar IntelliJ, crear un proyecto, ejecutar y depurar | Qué hace el compilador |

-----

## El mapa completo en una mirada

```
Introducción        ->  "¿qué es Kotlin y cómo se ve un programa?"
   fun main() { ... }, println / print, comentarios, errores

Del editor a salida ->  "¿qué pasa cuando presiono Run?"
   código .kt -> compilador (kotlinc) -> bytecode .class -> JVM -> salida

Entorno IntelliJ    ->  "¿dónde escribo y depuro mi código?"
   proyecto + JDK, Run (flecha verde), Debug con breakpoints
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

Ya sabes escribir y ejecutar un programa. Ahora aprendes a guardar datos en él: [Tipos y variables](../02-tipos-y-variables/README.md).
