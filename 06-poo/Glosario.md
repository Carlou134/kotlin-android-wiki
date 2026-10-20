# Glosario

Las palabras que aparecen en las lecciones de esta carpeta, explicadas de forma simple. Están en orden alfabético.

-----

**Bloque `init`.** Bloque de código que se ejecuta cada vez que se crea una instancia.

**Clase.** Plano que define las propiedades y funciones de un tipo de objeto.

**Constructor primario.** Parámetros entre paréntesis después del nombre de la clase; definen los valores de cada instancia al crearla.

**`copy()`.** Función de las data classes que crea una copia, permitiendo cambiar algunas propiedades.

**Data class.** Clase marcada con `data` cuyo objetivo principal es guardar datos.

**Desestructuración.** Extraer varias propiedades de un objeto en variables separadas: `val (a, b) = objeto`.

**Encapsulamiento.** Proteger el estado interno de un objeto para que solo se pueda cambiar a través de sus funciones.

**`entries`.** Propiedad de un enum que devuelve todas sus opciones, en orden de declaración.

**Enum.** Clase con un conjunto fijo de valores posibles, declarada con `enum class`.

**`field`.** Dentro de un getter o setter, el valor realmente guardado de la propiedad (backing field).

**Función miembro.** Función declarada dentro de una clase. Se llama sobre una instancia: `auto.drive()`.

**Getter.** Función que se ejecuta al leer una propiedad.

**`hashCode()`.** Número que representa el contenido de un objeto. Lo usan colecciones como sets y maps para buscar rápido.

**Instancia (objeto).** Un objeto concreto creado a partir de una clase.

**`internal`.** Visible desde cualquier lugar del mismo módulo (el mismo proyecto de Gradle).

**Invariante.** Regla que un objeto siempre debe cumplir, como "el saldo nunca es negativo".

**Modificador de visibilidad.** Palabra que decide desde dónde se puede usar algo: `public`, `private`, `protected` o `internal`.

**Notación de punto.** Forma de acceder a propiedades y funciones de un objeto: `objeto.propiedad`.

**`private`.** Visible solo dentro de la clase (o del archivo, si se declara fuera de una clase).

**`private set`.** Deja leer una propiedad desde afuera, pero solo modificarla desde adentro de la clase.

**Propiedad.** Variable que pertenece a una clase.

**Propiedad calculada.** Propiedad sin valor guardado cuyo getter calcula el resultado en cada lectura.

**`protected`.** Visible dentro de la clase y de sus clases hijas.

**`public`.** Visible desde cualquier lugar. Es el valor por defecto en Kotlin.

**Setter.** Función que se ejecuta al asignar un valor a una propiedad.

**`toString()`.** Función que convierte un objeto en texto. En una data class muestra sus propiedades.
