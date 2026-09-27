# Glosario

Las palabras que aparecen en las lecciones de esta carpeta, explicadas de forma simple. Están en orden alfabético.

-----

**Alcance (scope).** Zona del programa donde una variable existe. Una variable declarada dentro de una función no existe fuera de ella.

**Argumento.** Dato que le pasas a una función al llamarla.

**Argumento nombrado.** Argumento que indicas por su nombre al llamar: `saludar(name = "Ana")`.

**Argumento predeterminado.** Valor que toma un parámetro si no le pasas ninguno: `name: String = "Customer"`.

**Caso base.** Condición que detiene la recursión y devuelve un valor sin volver a llamar a la función.

**Función.** Bloque de código reutilizable con nombre, que se ejecuta cuando lo llamas.

**Función de expresión única.** Función de una sola expresión escrita con `=` y sin llaves: `fun cuadrado(n: Int) = n * n`.

**Lambda.** Función sin nombre escrita entre llaves que se puede guardar en una variable o pasar como argumento: `{ a: Int -> a * 2 }`.

**Llamada recursiva.** Cuando la función se llama a sí misma con nuevos argumentos.

**Recursión.** Técnica en la que una función se llama a sí misma hasta cumplir una condición.

**Stack overflow (desbordamiento de pila).** Error que ocurre cuando se acumulan demasiadas llamadas a funciones sin terminar.

**`tailrec`.** Modificador para funciones recursivas de cola: el compilador las convierte en un bucle y evita el stack overflow.

**Tipo de retorno.** Tipo del valor que devuelve una función, escrito después de `:` en su encabezado.
