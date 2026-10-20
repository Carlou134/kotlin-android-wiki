# Glosario

Las palabras que aparecen en las lecciones de esta carpeta, explicadas de forma simple. Están en orden alfabético.

-----

**`also`.** Función de alcance que recibe el objeto como `it` y devuelve el mismo objeto. Para efectos secundarios.

**`apply`.** Función de alcance que recibe el objeto como `this` y devuelve el mismo objeto. Para configurar.

**Borrado de tipos (type erasure).** En la JVM, los tipos genéricos desaparecen al ejecutar: una `List<String>` es solo una `List`.

**Callback.** Función que se pasa como argumento para que otra la ejecute cuando ocurra algo, como `onSuccess` u `onError`.

**Callback hell.** Callbacks anidados dentro de callbacks hasta que el código se vuelve ilegible. Se evita con corrutinas o resultados `sealed`.

**Closure.** Una lambda que recuerda las variables del lugar donde se creó.

**`crossinline`.** Marca una lambda de una función `inline` que se ejecuta en otro contexto; prohíbe el `return` no local.

**DSL.** Mini lenguaje dentro de Kotlin, armado con lambdas con receptor, como `routing { }` o `module { }`.

**Efecto secundario.** Algo que una función hace además de devolver un valor: imprimir, guardar, registrar en el log.

**Etiqueta (label).** Nombre que marca una lambda para hacer `return@nombre` desde adentro, como `return@forEach`.

**`find`.** Devuelve el primer elemento que cumple una condición, o `null` si ninguno cumple. Es lo mismo que `firstOrNull`; `first` en cambio lanza una excepción.

**`forEachIndexed`.** Como `forEach`, pero la lambda recibe dos parámetros: la posición y el elemento.

**Función anónima.** Función sin nombre propio; una lambda es una función anónima.

**Función de alcance.** Función de la librería estándar que ejecuta un bloque con un objeto como contexto: `let`, `run`, `with`, `apply`, `also`.

**Función de orden superior.** Función que recibe otra función como parámetro o la devuelve.

**`inline`.** Modificador que hace que el compilador copie el cuerpo de la función y de sus lambdas en cada llamada.

**`it`.** Nombre implícito del único parámetro de una lambda cuando no le pones nombre: `filter { it.total > 0 }` es `filter { pedido -> pedido.total > 0 }`. No es una palabra reservada, y no existe si la lambda tiene cero o dos o más parámetros.

**Lambda con receptor.** Lambda en la que `this` es un objeto de un tipo dado, como `Menu.() -> Unit`.

**`let`.** Función de alcance que recibe el objeto como `it` y devuelve el resultado del bloque. Típica con `?.`.

**`noinline`.** Marca una lambda de una función `inline` para que no se copie y se pueda guardar.

**Objeto de contexto.** El objeto sobre el que se ejecuta una función de alcance; se accede como `this` o como `it`.

**Referencia a función (`::`).** Usar una función existente como valor, sin llamarla: `pedidos.map(Pedido::id)`.

**`reified`.** Modificador que hace que un tipo genérico de una función `inline` siga disponible en tiempo de ejecución.

**`return` no local.** Un `return` dentro de una lambda inline que sale de la función que la contiene.

**`run`.** Función de alcance que recibe el objeto como `this` y devuelve el resultado del bloque. También existe sin receptor.

**Secuencia (`Sequence`).** Colección perezosa que procesa elemento por elemento, sin crear listas intermedias.

**`takeIf` / `takeUnless`.** Devuelven el objeto si cumple (o no cumple) una condición, y `null` si no.

**Tipo de función.** La forma de una función: `(String) -> Boolean` recibe un `String` y devuelve un `Boolean`.

**Trailing lambda.** Escribir la última lambda de una llamada fuera de los paréntesis.

**`Unit`.** El tipo de las funciones que no devuelven nada útil: `() -> Unit` no recibe ni devuelve nada.

**`with`.** Función de alcance que recibe el objeto como parámetro, lo usa como `this` y devuelve el resultado del bloque.
