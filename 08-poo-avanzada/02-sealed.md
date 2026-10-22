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

## `sealed class`: estado compartido entre los casos

Una `sealed class` puede tener **constructor**, así que cada caso le pasa un valor común. Ejemplo: los estados por los que pasa un email, cada uno con un `statusId`:

```kotlin
sealed class EmailStatus(val statusId: Int) {
  data class Sent(val sentTo: String) : EmailStatus(0)
  data class Failed(val error: String) : EmailStatus(1)
  data object Draft : EmailStatus(2)
  data class Scheduled(val time: String) : EmailStatus(3)
}
```

* Cada caso trae **sus** datos (`sentTo`, `error`, `time`) y además le pasa su `statusId` al padre.
* `Draft` no lleva datos, así que es un `data object`.
* `statusId` vive en el padre: **cualquier** `EmailStatus` lo expone, sin comprobar el tipo.

```kotlin
fun mostrar(status: EmailStatus) {
  println("Estado #${status.statusId}")   // ✅ siempre disponible
  // println(status.sentTo)               ❌ no compila: no sabes si es un Sent

  when (status) {
    is EmailStatus.Sent -> println("Enviado a ${status.sentTo}")      // smart cast
    is EmailStatus.Failed -> println("Falló: ${status.error}")
    EmailStatus.Draft -> println("Borrador")
    is EmailStatus.Scheduled -> println("Programado para ${status.time}")
  }
}
```

Fuera del `when` solo ves lo **común**. Dentro de cada rama `is`, el smart cast te abre los datos propios de ese caso.

Un detalle sobre `is` con un `data object`: `is EmailStatus.Draft` también compila, pero como hay una sola instancia, compararlo directo (`EmailStatus.Draft ->`) es más claro.

## `sealed interface`: un contrato que cumplen todos los casos

Una `sealed interface` no tiene constructor, pero sí puede exigir **propiedades** que cada caso implementa. Ejemplo: las acciones que un usuario puede hacer sobre su bandeja de entrada:

```kotlin
sealed interface EmailAction {
  val actionType: String

  data class Send(val recipient: String, val subject: String) : EmailAction {
    override val actionType = "send"
  }
  data class Delete(val emailId: String) : EmailAction {
    override val actionType = "delete"
  }
  data object SaveDraft : EmailAction {
    override val actionType = "draft"
  }
  data object Refresh : EmailAction {
    override val actionType = "refresh"
  }
}
```

La misma regla de siempre: `data class` si la acción lleva datos, `data object` si no.

### `when` para transformar un tipo en otro

Un uso muy común de `when` es **traducir** una entrada sin tipo (un número leído por consola, un string de una API) a un caso tipado:

```kotlin
fun leerAccion(comando: Int, emailId: String, destinatario: String, asunto: String): EmailAction = when (comando) {
  1 -> EmailAction.Send(destinatario, asunto)
  2 -> EmailAction.Delete(emailId)
  3 -> EmailAction.Refresh
  4 -> EmailAction.SaveDraft
  else -> throw IllegalArgumentException("Comando $comando no existe")
}
```

Acá el `else` **sí** es obligatorio, y está bien: el `when` es sobre un `Int`, que tiene infinitos valores. La regla de "evita el `else`" aplica cuando el sujeto del `when` es el tipo sellado.

### De una acción a un estado de pantalla

Un patrón que vas a ver muchísimo en Android: la **acción** del usuario produce un **estado** de UI.

```kotlin
sealed interface UiState {
  data object Loading : UiState
  data object Success : UiState
  data class Error(val mensaje: String) : UiState
}

fun procesar(action: EmailAction): UiState {
  println("Ejecutando: ${action.actionType}")   // común a todas: va FUERA del when

  return when (action) {
    is EmailAction.Send -> UiState.Success
    is EmailAction.Delete -> UiState.Error("No hay conexión")   // simulamos un fallo del backend
    EmailAction.Refresh -> UiState.Success
    EmailAction.SaveDraft -> UiState.Success
  }
}
```

Fíjate en el `println`: como `actionType` es parte del contrato, no hace falta repetirlo en cada rama. Lo que aplica a **todos** los casos va fuera del `when`, una sola vez.

(En una app real, `Refresh` emitiría primero `Loading` y después `Success` cuando termine. Para emitir varios estados en el tiempo hace falta `Flow`, que llega con las corrutinas.)

## `sealed class` vs `sealed interface`

| | `sealed class` | `sealed interface` |
| --- | --- | --- |
| Puede tener constructor y estado compartido | Sí | No (solo propiedades que cada caso implementa) |
| Un caso puede ser además parte de otra jerarquía | No (una sola clase padre) | Sí (muchas interfaces) |

Regla técnica: usa **`sealed interface`**, salvo que los casos necesiten compartir propiedades **con valor** a través de un constructor (como `statusId`).

Vas a escuchar también una convención: *"sealed class para estados de UI, sealed interface para acciones"*. Es una costumbre de muchos equipos, no una regla del lenguaje: un `UiState` como el de arriba funciona perfecto como `sealed interface`. Lo que decide es si necesitas estado compartido.

Dónde se declaran los casos: desde Kotlin 1.5, pueden estar en **cualquier archivo del mismo módulo y paquete**, no solo en el mismo archivo. Anidarlos dentro del padre (`EmailAction.Send`) es lo más común porque los agrupa bajo un nombre.

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

## Ejercicio: las carpetas del correo

Modela las carpetas de un cliente de correo con una `sealed class EmailFolder` que tenga una propiedad compartida `count: Int`:

1. `Inbox`, sin datos propios, con `count = 10`.
2. `Spam`, con un `senderDomain: String`, con `count = 5`.
3. `Sent`, sin datos propios, con `count = 3`.

Después escribe `fun describir(folder: EmailFolder): String` con un `when` sin `else`.

<details>
<summary>Ver solución</summary>

```kotlin
sealed class EmailFolder(val count: Int) {
  data object Inbox : EmailFolder(10)
  data class Spam(val senderDomain: String) : EmailFolder(5)
  data object Sent : EmailFolder(3)
}

fun describir(folder: EmailFolder): String {
  val detalle = when (folder) {
    EmailFolder.Inbox -> "Bandeja de entrada"
    is EmailFolder.Spam -> "Spam de ${folder.senderDomain}"
    EmailFolder.Sent -> "Enviados"
  }
  return "$detalle (${folder.count})"   // count es común: fuera del when
}
```

¿Por qué `sealed class` y no `sealed interface`? Porque `count` es un valor que cada caso le pasa al padre por constructor. Con una interfaz, cada caso tendría que escribir su propio `override val count`.

</details>

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
5. `sealed class` comparte estado por constructor (`statusId`); `sealed interface` comparte un contrato (`actionType`) y es la opción por defecto. Úsalas para resultados, errores, acciones y estados de UI.

-----

## Siguiente lección

[3. Extension functions](03-extension-functions.md): agregarle funciones a clases que no son tuyas.
