# kotlinx.serialization

## En una frase

kotlinx.serialization convierte objetos de Kotlin a JSON y de vuelta; un plugin del compilador genera el código de conversión para cada clase marcada con `@Serializable`, así que los errores se detectan al compilar y no hace falta reflexión.

-----

## Antes de empezar

Conviene que ya sepas:

* Data classes, valores por defecto y nulables: [Data classes](../06-poo/03-data-classes.md) y [Nulabilidad](../07-null-safety-y-errores/01-nulabilidad.md).
* `sealed`, `value class` y `object`: [Clases e interfaces selladas](../08-poo-avanzada/02-sealed.md) y [`value class`, `object` y `companion object`](../08-poo-avanzada/04-value-class-y-object.md).

Palabras nuevas (también están en el [Glosario](Glosario.md)):

* **Serializar:** Convertir un objeto a un formato de texto o bytes (JSON).
* **Deserializar:** Convertir de vuelta, del formato al objeto.
* **Serializador:** El código que sabe convertir un tipo concreto.
* **Plugin del compilador:** Extensión del compilador de Kotlin que genera código durante la compilación.
* **Polimorfismo (en JSON):** Guardar en el JSON qué subtipo concreto es un objeto, para reconstruirlo bien.
* **Discriminador:** El campo del JSON que dice qué subtipo es, normalmente `"type"`.

-----

## El problema: JSON escrito a mano

Sin una librería, convertir a JSON es escribir strings a mano, y leerlo es buscar campos en un mapa de `Any?` con casts por todos lados. Cada campo nuevo es una oportunidad de equivocarse.

Otras librerías (como Jackson o Gson) usan **reflexión**: en tiempo de ejecución inspeccionan la clase para saber qué campos tiene. Funciona, pero los errores aparecen al ejecutar, y no entienden bien la nulabilidad ni los valores por defecto de Kotlin.

kotlinx.serialization genera el código **al compilar**, y respeta las reglas de Kotlin.

## Configuración

En Gradle, el plugin del compilador y la librería:

```kotlin
plugins {
  kotlin("plugin.serialization") version "<versión de Kotlin>"
}

dependencies {
  implementation("org.jetbrains.kotlinx:kotlinx-serialization-json:<versión>")
}
```

Sin el plugin, `@Serializable` no hace nada y fallas en tiempo de ejecución con "Serializer for class X is not found".

## Lo básico

```kotlin
@Serializable
data class LineaResponse(val productoId: String, val cantidad: Int)

@Serializable
data class PedidoResponse(
  val id: String,
  val lineas: List<LineaResponse>,
  val nota: String? = null,
)

val pedido = PedidoResponse("p-1", listOf(LineaResponse("prod-1", 2)))

val texto = Json.encodeToString(pedido)
// {"id":"p-1","lineas":[{"productoId":"prod-1","cantidad":2}]}

val deVuelta = Json.decodeFromString<PedidoResponse>(texto)
```

Fíjate en que `nota` no aparece en el JSON: por defecto, los campos que tienen su **valor por defecto** no se escriben.

## Configurar `Json`

`Json` sin configurar es estricto. En un backend se suele crear **una** instancia configurada y compartirla:

```kotlin
val JsonApi = Json {
  ignoreUnknownKeys = true   // un campo que no conozco no rompe la lectura
  explicitNulls = false      // no escribe "campo": null; al leer, un campo ausente es null
  encodeDefaults = true      // escribe también los campos con valor por defecto
}
```

| Opción | Qué hace | Cuándo |
| --- | --- | --- |
| `ignoreUnknownKeys` | Ignora campos del JSON que tu clase no tiene | Siempre al leer APIs de terceros; casi siempre en general |
| `explicitNulls = false` | No escribe los `null`; los ausentes se leen como `null` | Respuestas más limpias |
| `encodeDefaults` | Escribe los campos aunque tengan su valor por defecto | Cuando el cliente necesita ver todos los campos |
| `coerceInputValues` | Si llega un valor inválido (por ejemplo, un enum desconocido) usa el valor por defecto | Para tolerar valores nuevos de un proveedor |
| `prettyPrint` | JSON con saltos de línea | Solo para depurar o para logs legibles |

La **misma** instancia se le pasa a Ktor, así las rutas y el resto del código serializan igual:

```kotlin
install(ContentNegotiation) { json(JsonApi) }
```

## Campos obligatorios y opcionales

```kotlin
@Serializable
data class CrearClienteRequest(
  val email: String,                 // obligatorio: si falta, error
  val nombre: String,                // obligatorio
  val telefono: String? = null,      // opcional: puede faltar o ser null
  val boletin: Boolean = false,      // opcional: si falta, false
)
```

La regla: **un campo sin valor por defecto es obligatorio**. Si falta en el JSON, la deserialización falla con una excepción (en Ktor, eso termina en un `400` gracias a StatusPages).

Un error común: `val telefono: String?` **sin** `= null`. Es nulable, pero sigue siendo obligatorio: el campo tiene que venir, aunque sea con `null` (salvo que uses `explicitNulls = false`).

## Nombres distintos: `@SerialName`

Las APIs de terceros suelen usar `snake_case` o nombres crípticos. Tu clase usa nombres de Kotlin y `@SerialName` hace la traducción:

```kotlin
@Serializable
data class DeviceDto(
  @SerialName("serial_number") val numeroDeSerie: String,
  @SerialName("battery_level") val bateria: Double,
  @SerialName("last_seen") val ultimaSenal: Long,
)
```

## Enums

Por defecto, un enum se serializa con su nombre (`"PENDIENTE"`). Si el JSON usa otros valores:

```kotlin
@Serializable
enum class EstadoEnvio {
  @SerialName("in_transit") EN_CAMINO,
  @SerialName("delivered") ENTREGADO,
}
```

Si un proveedor agrega un valor nuevo, la lectura falla. Con `coerceInputValues = true` y un valor por defecto en el campo, se usa el valor por defecto en lugar de fallar.

## `value class`

Una `value class` marcada `@Serializable` se escribe como el valor que envuelve, sin un objeto alrededor:

```kotlin
@Serializable
@JvmInline
value class PedidoId(val value: String)

@Serializable
data class Ref(val pedido: PedidoId)
// {"pedido":"p-1"}   y no {"pedido":{"value":"p-1"}}
```

## `sealed`: polimorfismo

```kotlin
@Serializable
sealed interface Evento {
  @Serializable @SerialName("pedido_creado")
  data class PedidoCreado(val pedidoId: String) : Evento

  @Serializable @SerialName("pedido_cancelado")
  data class PedidoCancelado(val pedidoId: String, val motivo: String) : Evento
}

val texto = JsonApi.encodeToString<Evento>(Evento.PedidoCancelado("p-1", "sin stock"))
// {"type":"pedido_cancelado","pedidoId":"p-1","motivo":"sin stock"}

val evento = JsonApi.decodeFromString<Evento>(texto)   // vuelve como PedidoCancelado
```

* El campo `"type"` es el **discriminador**: dice qué subtipo es.
* `@SerialName` en cada subtipo define el valor del discriminador. Sin él, se usa el nombre completo de la clase, que expone tus paquetes internos.
* El nombre del campo se cambia en la configuración, con `classDiscriminator = "tipo"`.

## Serializadores propios

Para tipos que la librería no conoce, como `BigDecimal`:

```kotlin
object BigDecimalComoTexto : KSerializer<BigDecimal> {
  override val descriptor = PrimitiveSerialDescriptor("BigDecimal", PrimitiveKind.STRING)

  override fun serialize(encoder: Encoder, value: BigDecimal) =
    encoder.encodeString(value.toPlainString())

  override fun deserialize(decoder: Decoder): BigDecimal =
    BigDecimal(decoder.decodeString())
}

@Serializable
data class PrecioResponse(
  @Serializable(with = BigDecimalComoTexto::class) val total: BigDecimal,
)
// {"total":"150.00"}
```

El dinero viaja como **string**, como viste en [Diseño REST](../13-http-y-apis/02-diseno-rest.md): un número JSON se lee como `double` en muchos clientes y pierde precisión.

## JSON sin forma fija

Si una parte del JSON no tiene estructura conocida (por ejemplo, un campo `"extra"` libre de un proveedor), usa `JsonElement`:

```kotlin
@Serializable
data class WebhookDto(val evento: String, val datos: JsonObject)

val bateria = webhook.datos["battery"]?.jsonPrimitive?.doubleOrNull
```

Úsalo solo en el borde, y conviértelo a tipos propios lo antes posible.

## Dónde va `@Serializable`

**Solo en los DTOs** de los adapters (`*Request`, `*Response`, los DTOs de proveedores), nunca en las entidades de dominio. Si el dominio es `@Serializable`, su forma interna se convierte en tu API sin que nadie lo decida. Es la regla que verificas con Konsist en [Tests de arquitectura](../12-arquitectura/06-tests-de-arquitectura.md).

-----

## Resumen en 5 líneas

1. kotlinx.serialization genera los serializadores al compilar con un plugin: `@Serializable` en la clase y `Json.encodeToString` / `decodeFromString<T>`.
2. Una instancia de `Json` configurada (`ignoreUnknownKeys`, `explicitNulls`) se comparte en todo el backend, incluido Ktor.
3. Sin valor por defecto, un campo es obligatorio aunque sea nulable; `@SerialName` traduce nombres de terceros.
4. Las `value class` se escriben como su valor; los `sealed` usan un discriminador `"type"`, con `@SerialName` en cada subtipo.
5. Serializadores propios para tipos como `BigDecimal`; `@Serializable` solo en los DTOs, nunca en el dominio.

-----

## Siguiente lección

[2. Tiempo con `Instant` y kotlinx.datetime](02-tiempo-con-instant.md): fechas y horas sin errores de zona horaria.
