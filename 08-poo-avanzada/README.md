# POO avanzada

En esta carpeta aprendes las herramientas de Kotlin para **modelar con tipos**: contratos que varias clases cumplen (interfaces), jerarquías cerradas que el compilador conoce de memoria (`sealed`), funciones que agregas a clases que no son tuyas (extension functions) y tipos que protegen tus datos sin costo (`value class`), además de los objetos únicos (`object` y `companion object`).

-----

## Antes de empezar

Conviene que ya hayas leído [Programación orientada a objetos](../06-poo/README.md), sobre todo [Clases](../06-poo/01-clases.md), [Data classes](../06-poo/03-data-classes.md) y [Enums](../06-poo/04-enums.md), y [Null safety y errores](../07-null-safety-y-errores/README.md).

Si en algún momento aparece una palabra que no conoces, búscala en el [Glosario](Glosario.md).

-----

## Orden de lectura

Las lecciones se apoyan una en la otra, así que conviene leerlas en este orden:

| Lección | Qué aprendes | Qué necesitas saber antes |
| --- | --- | --- |
| [1. Herencia e interfaces](01-herencia-e-interfaces.md) | `open`, `abstract`, `override`, interfaces, polimorfismo y `fun interface` | Clases |
| [2. Clases e interfaces selladas](02-sealed.md) | `sealed class`, `sealed interface`, `data object` y `when` exhaustivo | Herencia, interfaces y enums |
| [3. Extension functions](03-extension-functions.md) | Agregar funciones y propiedades a clases ajenas, y cómo se resuelven | Funciones y clases |
| [4. `value class`, `object` y `companion object`](04-value-class-y-object.md) | Tipos que envuelven un valor, singletons y funciones "de la clase" | Data classes |

-----

## El mapa completo en una mirada

```
Herencia e interfaces  ->  "un contrato, muchas implementaciones"
   interface Notificador { fun enviar(mensaje: String) }
   class EmailNotificador : Notificador { override fun enviar(...) { ... } }
   val notificadores: List<Notificador> = listOf(EmailNotificador(), SmsNotificador())

Sealed                 ->  "una lista cerrada de casos, cada uno con sus datos"
   sealed interface Resultado {
     data class Exito(val valor: Int) : Resultado
     data object NoEncontrado : Resultado
   }
   when (r) { is Resultado.Exito -> ...; Resultado.NoEncontrado -> ... }  // sin else

Extension functions    ->  "agregarle funciones a una clase que no es tuya"
   fun String.esEmail() = contains("@")       "ana@mail.com".esEmail()

value class / object   ->  "tipos sin costo y objetos únicos"
   @JvmInline value class PedidoId(val value: String)
   object Configuracion { ... }               // una sola instancia
   companion object { fun desde(...) = ... }  // Pedido.desde(...)
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

Ya sabes modelar con tipos. Lo próximo es la [programación funcional](../09-funcional/README.md): lambdas, funciones de alcance e `inline`. Vuelve al índice: [README principal](../README.md).
