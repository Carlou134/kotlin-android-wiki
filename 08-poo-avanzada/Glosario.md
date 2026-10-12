# Glosario

Las palabras que aparecen en las lecciones de esta carpeta, explicadas de forma simple. Están en orden alfabético.

-----

**`abstract`.** Clase que no se puede instanciar, o función sin cuerpo que las clases hijas deben implementar.

**Boxing.** Cuando Kotlin sí crea el objeto envoltorio de una `value class` (por ejemplo, al usarla como nulable o dentro de genéricos).

**`companion object`.** Objeto dentro de una clase cuyos miembros se usan con el nombre de la clase. Reemplaza a `static`.

**`data object`.** Un objeto único con un `toString()` legible, ideal para los casos sin datos de un tipo sellado.

**Extension function.** Función declarada fuera de una clase que se llama como si fuera miembro de ella.

**Extension property.** Propiedad calculada agregada a una clase desde afuera. No puede guardar valores.

**`fun interface`.** Interfaz con una sola función abstracta, que se puede implementar con una lambda.

**Función de fábrica.** Función que crea instancias, en vez de llamar al constructor directamente; suele validar antes de crear.

**Herencia.** Mecanismo por el que una clase hija recibe las propiedades y funciones de una clase padre.

**Interfaz.** Contrato con funciones (y propiedades) que una clase se compromete a implementar.

**Jerarquía sellada.** Conjunto cerrado de subtipos declarado con `sealed`; no se pueden agregar otros desde afuera.

**`object`.** Declaración de Kotlin que crea un singleton.

**`open`.** Marca una clase o función para que se pueda heredar o sobrescribir. En Kotlin, por defecto, todo es `final`.

**`override`.** Palabra clave para reemplazar una función o propiedad heredada.

**Polimorfismo.** Usar objetos de clases distintas a través de un mismo tipo, y que cada uno responda a su manera.

**Receptor (receiver).** El tipo al que se le agrega una extensión; dentro de la función, es `this`.

**Resolución estática.** La extensión que se ejecuta se decide por el tipo declarado de la variable, al compilar, no por el objeto real.

**Singleton.** Un tipo del que existe una sola instancia en todo el programa.

**Smart cast.** Cuando Kotlin sabe el tipo concreto de una variable después de comprobarlo con `is`, y te deja usarla como ese tipo.

**`value class`.** Clase que envuelve un único valor y, en la mayoría de los casos, no crea un objeto extra en tiempo de ejecución.

**`when` exhaustivo.** Un `when` que cubre todos los casos posibles de un tipo sellado o enum, sin necesitar `else`.
