# Extension functions

## En una frase

Una extension function agrega una función a una clase **sin modificarla ni heredar de ella**: la declaras afuera, y se usa con notación de punto como si fuera propia.

-----

## Antes de empezar

Conviene que ya sepas:

* Declarar funciones: [Funciones](../05-funciones/01-funciones.md).
* Tipos nulables: [Nulabilidad](../07-null-safety-y-errores/01-nulabilidad.md).

Palabras nuevas (también están en el [Glosario](Glosario.md)):

* **Extension function:** Función declarada fuera de una clase que se llama como si fuera miembro de ella.
* **Receptor (receiver):** El tipo al que se le agrega la extensión; dentro de la función, es `this`.
* **Extension property:** Propiedad calculada agregada a una clase desde afuera.
* **Resolución estática:** La extensión que se ejecuta se decide por el tipo declarado de la variable, al compilar.

-----

## El problema: clases que no puedes tocar

Quieres saber si un `String` parece un email. `String` es de la librería estándar: no puedes agregarle funciones.

La solución clásica es una clase de utilidades:

```kotlin
object StringUtils {
  fun esEmail(texto: String) = texto.contains("@") && texto.contains(".")
}

StringUtils.esEmail(correo)
```

Funciona, pero se lee al revés: primero la utilidad, después el dato.

## La solución: extender el tipo

```kotlin
fun String.esEmail(): Boolean = contains("@") && contains(".")

"ana@mail.com".esEmail()  // true
correo.esEmail()
```

* `String.` antes del nombre es el **receptor**: el tipo al que se le agrega la función.
* Dentro, `this` es el `String` sobre el que se llamó. `contains("@")` es `this.contains("@")`.

## Con parámetros y con tus propias clases

```kotlin
data class Pedido(val subtotal: Double, val pais: String)

fun Pedido.totalConImpuestos(tasa: Double): Double = subtotal * (1 + tasa)

val p = Pedido(subtotal = 100.0, pais = "AR")
println(p.totalConImpuestos(0.21)) // 121.0
```

¿Por qué extender tu propia clase en vez de poner la función adentro? Para mantener la clase enfocada. `Pedido` modela un pedido; cómo se **muestra** o cómo se **convierte** a otro formato puede vivir en otro archivo, cerca de quien lo necesita.

## Extension properties

También puedes agregar propiedades **calculadas** (sin estado guardado):

```kotlin
val String.iniciales: String
  get() = split(" ").filter { it.isNotEmpty() }.joinToString("") { it.first().uppercase() }

println("ana lopez".iniciales) // AL
```

Una extension property no puede guardar un valor: no tiene dónde, porque la clase no se modificó.

## Receptor nulable

Puedes extender un tipo nulable y manejar el `null` adentro:

```kotlin
fun String?.oVacio(): String = this ?: ""

val apodo: String? = null
println(apodo.oVacio().length) // 0
```

La librería estándar hace esto con `isNullOrBlank()`: por eso puedes llamarla sobre un `String?` sin `?.`.

## Cómo se resuelven: la trampa

Las extensiones **no modifican la clase**. El compilador elige cuál llamar según el **tipo declarado** de la variable, no según el objeto real:

```kotlin
open class Animal
class Perro : Animal()

fun Animal.sonido() = "..."
fun Perro.sonido() = "Guau"

val a: Animal = Perro()
println(a.sonido()) // "..."  ← usa la extensión de Animal, porque `a` está declarado como Animal
```

Con funciones miembro y `override`, se habría ejecutado la del `Perro`. Con extensiones, no hay polimorfismo.

Además, si la clase ya tiene una función **miembro** con el mismo nombre y parámetros, **gana el miembro** y tu extensión se ignora.

## Visibilidad

Las extensiones tienen la visibilidad que les des:

```kotlin
private fun Pedido.esGrande() = subtotal > 10_000  // solo visible en este archivo
```

Una extensión `private` es perfecta para traducciones que nadie más debería usar.

## El uso estrella: convertir entre modelos

En Kotlin, en vez de clases `PedidoMapper` con funciones `toDto(pedido)`, se usan extensiones:

```kotlin
fun Pedido.toResponse() = PedidoResponse(total = subtotal, pais = pais)
fun CrearPedidoRequest.toPedido() = Pedido(subtotal = monto, pais = codigoPais)

val respuesta = pedido.toResponse()
```

Se lee natural ("pedido a respuesta"), no necesita instanciar nada, y puede vivir justo en el archivo que hace la conversión.

## Cuándo no usarlas

* **Para lógica central de una clase tuya:** si la función usa el estado interno y es parte de lo que la clase **es**, va adentro como miembro.
* **Para "todo":** cientos de extensiones sobre `String` repartidas en el proyecto vuelven imposible saber qué existe. Agrúpalas por tema y mantenlas cerca de su uso.
* **Cuando necesitas polimorfismo:** las extensiones no se sobrescriben.

## Dónde lo vas a ver

* `ResultRow.toPedido()` dentro de los repositorios: [Exposed DSL](../15-persistencia/04-exposed-dsl.md).
* `Pedido.toResponse()` en las rutas: [Arquitectura hexagonal](../12-arquitectura/02-arquitectura-hexagonal.md).
* `Route.pedidoRoutes()` y `Application.configurarX()`: [Ktor server y plugins](../14-ktor-y-koin/01-ktor-server-y-plugins.md).

-----

## Resumen en 5 líneas

1. Una extension function agrega una función a un tipo sin modificarlo: `fun String.esEmail() = ...`.
2. Dentro, `this` es el receptor; también existen extension properties (solo calculadas) y receptores nulables.
3. Se resuelven por el tipo declarado, sin polimorfismo; si existe un miembro con la misma firma, gana el miembro.
4. Su uso estrella es convertir entre modelos (`toResponse()`, `toPedido()`), en lugar de clases `*Mapper`.
5. No las uses para la lógica central de tus clases ni las desparrames sin orden.

-----

## Siguiente lección

[4. `value class`, `object` y `companion object`](04-value-class-y-object.md): tipos sin costo y objetos únicos.
