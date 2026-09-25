# Recursion

Aprende cómo crear funciones recursivas en Kotlin.

## **¿Qué es la recursión?**

La recursión es el proceso en el cual una función se llama a sí misma hasta que se cumple una condición. Este concepto es similar a un bucle `while`, en el que un bloque de código se ejecutará repetidamente hasta que se cumpla una condición.

La recursión es un tema avanzado en lo que respecta a funciones y programación; a menudo se necesita tiempo y práctica hasta que un programador se sienta cómodo creando y usando funciones recursivas, así que no te desanimes si no todo se asimila de inmediato.

**Desglosando una función recursiva**

En una función recursiva, se crea un **caso base** (generalmente una expresión `if`) para verificar una condición. Si la condición es verdadera, la función devolverá un valor. Si la condición es falsa, la función devolverá una llamada a sí misma con un nuevo conjunto de valores de argumentos; esto se conoce como **llamada recursiva**. La función continuará llamándose a sí misma hasta que se alcance el caso base.

Usemos una función recursiva para obtener la suma de un grupo de números desde `num` hasta 0:

```kotlin
fun recursiveSum(num: Int): Int {
  // caso base
  if (num == 0) {
    return num
  } else { 
    // llamada recursiva
    return num + recursiveSum(num - 1)
  }
}
```

Si llamáramos a esta función con un argumento de `4`, obtendríamos un valor de retorno de `10`. Intentemos desglosar cómo funcionó. Hasta que el parámetro `num` tenga un valor de `0`, se ejecutará el código en el bloque `else`, `return num + recursiveSum(num - 1)`, de la siguiente manera:

```
recursiveSum(4) devuelve 4 + recursiveSum(3)
recursiveSum(3) devuelve 3 + recursiveSum(2)
recursiveSum(2) devuelve 2 + recursiveSum(1)
recursiveSum(1) devuelve 1 + recursiveSum(0)
```

Una vez que llamamos a `recursiveSum(0)`, se alcanza el caso base porque `num` es `0`. Ahora, necesitamos devolver el valor de cada llamada, comenzando desde la última llamada realizada:

```
recursiveSum(0) devuelve 0
recursiveSum(1) devuelve 1 + 0, que es igual a 1
recursiveSum(2) devuelve 2 + 1, que es igual a 3
recursiveSum(3) devuelve 3 + 3, que es igual a 6
recursiveSum(4) devuelve 4 + 6, que es igual a 10
```

**Nota:** Al igual que un bucle `while`, una función recursiva puede ejecutarse infinitamente si no creamos un caso base que eventualmente detenga las llamadas recursivas.

----

## **Funciones recursivas de cola (Tail-Recursive)**

Las funciones recursivas corren el riesgo de **desbordamiento de pila (stack overflow)**. El desbordamiento de pila ocurre en funciones recursivas cuando se está usando demasiados datos para almacenar los valores creados en cada llamada recursiva y puede provocar que nuestro programa falle.

Afortunadamente para nosotros, Kotlin tiene una solución para esto: las **funciones recursivas de cola** (tail-recursive functions). Cuando se usa una función recursiva de cola, el compilador puede optimizar el código para prevenir el desbordamiento de pila.

Recreemos la función `recursiveSum()` anterior. Para declarar una función recursiva de cola, anteponemos `tailrec` a la palabra clave `fun` en el encabezado:

```kotlin
tailrec fun recursiveSum(num: Int, sum: Int = 0): Int {
  return if (num == 0) sum 
  else recursiveSum(num - 1, sum + num)
}

fun main() {
  println(recursiveSum(num = 10)) // Imprime: 55
}
```

Agregamos un segundo argumento, `sum`, para almacenar el valor de retorno final. Para que una función recursiva sea considerada para recursión de cola, la última operación en la función debe ser una llamada a sí misma. En este caso, la última instrucción es `recursiveSum(num - 1, sum + num)`.

El código `return if (num == 0) sum` significa que el caso base es `num == 0` y el valor de retorno es `sum`. El código `else recursiveSum(num - 1, sum + num)` es la llamada recursiva que cambia el argumento `num` disminuyéndolo en 1, mientras suma el valor de `num` al valor de retorno final `sum`.

---

## **Para finalizar**

Excelente trabajo por llegar al final de este artículo. Repasemos lo que aprendimos:

- La **recursión** es el proceso de una función que se llama a sí misma dentro del bloque de código de la función.
- Toda función recursiva necesita un **caso base** y una **llamada recursiva**.
- El **caso base** define qué condición debe cumplirse para salir de la función recursiva, mientras que la **llamada recursiva** se usa para llamar a la función con un nuevo conjunto de argumentos.
- Una **función recursiva de cola** es una forma más eficiente de producir una función recursiva porque previene el desbordamiento de pila (stack overflow).

La recursión es uno de los temas más avanzados de este curso; no te desanimes si el concepto no se asimiló de inmediato. La recursión no es un concepto de programación fundamental, pero una vez que le tomas el truco, puede ser una herramienta valiosa para proyectos de codificación, preguntas de entrevistas y más.

----