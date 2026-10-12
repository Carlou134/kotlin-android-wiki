# Clases e interfaces selladas (`sealed`)

## En una frase

Una jerarquía `sealed` es una lista **cerrada** de subtipos que el compilador conoce completa; cada caso puede llevar sus propios datos, y un `when` sobre ella te obliga a manejar todos los casos.

-----

## Antes de empezar

Conviene que ya sepas:

* Enums y `when`: [Enums](../06-poo/04-enums.md).
* Herencia e interfaces: [Herencia e interfaces](01-herencia-e-interfaces.md).

Palabras nuevas (también están en el [Glosario](Glosario.md)):

* **Jerarquía sellada:** Conjunto cerrado de subtipos; no se pueden agregar otros desde afuera del módulo y del paquete donde se declaró.
* **`when` exhaustivo:** Un `when` que cubre todos los casos posibles, sin necesitar `else`.
* **Smart cast:** Cuando Kotlin sabe el tipo concreto de una variable después de comprobarlo con `is`, y te deja usarla como ese tipo.
* **`data object`:** Un objeto único con un `toString()` legible, ideal para casos sin datos.

-----

## El problema: los enums no llevan datos distintos

Quieres modelar el resultado de un pago. Con un enum:

```kotlin
enum class ResultadoPago { APROBADO, RECHAZADO, PENDIENTE }
```

Pero cada caso necesita **datos distintos**:

* `APROBADO` necesita el código de autorización.
* `RECHAZADO` necesita el motivo.
* `PENDIENTE` no necesita nada.

En un enum, todas las constantes tienen las **mismas** propiedades. Podrías poner todo como nulable (`codigo: String?`, `motivo: String?`), pero entonces nada te impide crear un "aprobado con motivo de rechazo".

## La solución: `sealed interface`

```kotlin
sealed interface ResultadoPago {
  data class Aprobado(val codigoAutorizacion: String) : ResultadoPago
  data class Rechazado(val motivo: String) : ResultadoPago
  data object Pendiente : ResultadoPago
}
```

* Cada caso es una clase (o un objeto) con **sus** datos.
* `Pendiente` no tiene datos, así que es un `data object`: hay una sola instancia.
* Nadie puede agregar un cuarto caso desde otro módulo. La lista está **cerrada**.

## `when` exhaustivo

Como el compilador conoce **todos** los casos, un `when` usado como expresión no necesita `else`:

```kotlin
fun mensaje(resultado: ResultadoPago): String = when (resultado) {
  is ResultadoPago.Aprobado -> "Pago aprobado: ${resultado.codigoAutorizacion}"
  is ResultadoPago.Rechazado -> "Pago rechazado: ${resultado.motivo}"
  ResultadoPago.Pendiente -> "Pago pendiente"
}
```

Fíjate en el **smart cast**: dentro de la rama `is ResultadoPago.Aprobado`, Kotlin ya sabe que `resultado` es un `Aprobado`, así que puedes leer `codigoAutorizacion` sin convertir nada.

Para `data object` no hace falta `is`: como hay una sola instancia, se compara directamente.

## La gran ventaja: agregar un caso no compila

Mañana el negocio agrega pagos en revisión manual:

```kotlin
sealed interface ResultadoPago {
  // ...
  data class EnRevision(val analista: String) : ResultadoPago
}
```

**Todos** los `when` que no manejan `EnRevision` dejan de compilar. El compilador te lleva de la mano a cada lugar del código que tienes que actualizar.

Con un `else`, en cambio, el caso nuevo cae en el `else` en silencio. Por eso, sobre un tipo `sealed`, **evita el `else`**: le quita al compilador la posibilidad de avisarte.

## `sealed class` vs `sealed interface`

```kotlin
sealed class Forma(val nombre: String) {
  class Circulo(val radio: Double) : Forma("círculo")
  class Cuadrado(val lado: Double) : Forma("cuadrado")
}
```

| | `sealed class` | `sealed interface` |
| --- | --- | --- |
| Puede tener constructor y estado compartido | Sí | No |
| Un caso puede ser además parte de otra jerarquía | No (una sola clase padre) | Sí (muchas interfaces) |

Regla práctica: usa **`sealed interface`**, salvo que todos los casos necesiten compartir propiedades con valor.

## ¿Enum o sealed?

| Usa enum si... | Usa sealed si... |
| --- | --- |
| Todos los casos tienen los mismos datos (o ninguno) | Cada caso tiene datos distintos |
| Necesitas recorrer todos los valores (`entries`) | Algunos casos tienen muchas instancias distintas |
| Ejemplo: `EstadoPedido { PENDIENTE, ENVIADO }` | Ejemplo: `ResultadoPago`, eventos, errores |

## Modelar errores y resultados

El uso más importante de `sealed`: devolver **todos** los resultados posibles de una operación, en vez de lanzar excepciones que nadie ve venir.

```kotlin
sealed interface BuscarClienteResultado {
  data class Encontrado(val cliente: Cliente) : BuscarClienteResultado
  data object NoExiste : BuscarClienteResultado
  data class Bloqueado(val desde: String) : BuscarClienteResultado
}

fun buscarCliente(id: String): BuscarClienteResultado { /* ... */ }
```

Mirando la firma, ya sabes qué puede pasar. Y quien la llama está **obligado** a manejar cada caso.

## Dónde lo vas a ver

* Los **outcomes** de los use cases: [La capa application](../12-arquitectura/05-capa-application.md).
* Los motivos de denegación: [RBAC y multi-tenancy](../16-seguridad/05-rbac-y-multitenancy.md).
* Los resultados de un compare-and-swap: [Bloqueos y concurrencia](../15-persistencia/06-bloqueos-y-concurrencia.md).

-----

## Resumen en 5 líneas

1. Una jerarquía `sealed` es una lista cerrada de subtipos que el compilador conoce completa.
2. A diferencia de un enum, cada caso puede llevar sus propios datos; los casos sin datos son `data object`.
3. Un `when` sobre un tipo sealed es exhaustivo sin `else`, y hace smart cast en cada rama.
4. Agregar un caso rompe la compilación en cada `when` que no lo maneja: por eso se evita el `else`.
5. Prefiere `sealed interface`; úsalo para modelar resultados y errores en vez de excepciones.

-----

## Siguiente lección

[3. Extension functions](03-extension-functions.md): agregarle funciones a clases que no son tuyas.
