# Tiempo con `Instant` y kotlinx.datetime

## En una frase

Un `Instant` es un punto exacto en la línea del tiempo, igual en todo el mundo; las fechas y horas "de calendario" (`LocalDate`, `LocalDateTime`) dependen de una zona horaria. El backend guarda y transmite instantes, y convierte a hora local solo cuando una regla de negocio lo necesita.

-----

## Antes de empezar

Conviene que ya sepas:

* Serializar con kotlinx.serialization: [kotlinx.serialization](01-kotlinx-serialization.md).
* Qué es un puerto y cómo se inyecta: [SOLID e inversión de dependencias](../12-arquitectura/01-solid-y-dip.md).

Palabras nuevas (también están en el [Glosario](Glosario.md)):

* **`Instant`:** Un punto exacto en el tiempo, independiente de cualquier zona horaria.
* **UTC:** El tiempo universal coordinado; la referencia contra la que se definen todas las zonas.
* **Zona horaria:** Las reglas de una región para pasar de UTC a la hora local, incluidos los cambios de horario de verano.
* **`LocalDate` / `LocalDateTime`:** Una fecha (o fecha y hora) de calendario, sin zona: "12 de octubre", "12 de octubre a las 8:00".
* **`Duration`:** Una cantidad de tiempo: 15 minutos, 2 horas.
* **`Clock`:** La fuente de la hora actual.
* **Horario de verano (DST):** Cambio estacional de la hora local en algunas regiones.

-----

## El problema: "las 10:00" no es un momento

Un usuario en Buenos Aires crea un pedido "a las 10:00". Otro en Madrid lo ve "a las 15:00". Es **el mismo momento**, escrito en dos zonas.

Si guardas `2026-10-12 10:00` sin zona, ese dato es ambiguo: ¿10:00 dónde? Cuando el servidor está en otra zona, o un cliente está de viaje, los reportes se corren horas y las alertas saltan a destiempo.

## Dos tipos de tiempo, dos tipos de dato

| Necesitas representar | Tipo | Ejemplo |
| --- | --- | --- |
| Cuándo pasó algo | `Instant` | Creación de un pedido, último login, última señal de un dispositivo |
| Una fecha de calendario | `LocalDate` | Cumpleaños, fecha de vencimiento de un contrato, feriado |
| Una hora de reloj de pared | `LocalTime` | "El reporte se envía a las 08:00" |
| Fecha y hora de calendario | `LocalDateTime` | Solo como paso intermedio, junto con una zona |

La regla: **eventos → `Instant`**. Casi todo lo que guarda un backend es un evento.

## `Instant` y `Clock`

```kotlin
val ahora: Instant = Clock.System.now()

val vence = ahora + 15.minutes          // aritmética con Duration
val haceUnDia = ahora - 1.days

val transcurrido: Duration = ahora - pedido.creadoEn
if (transcurrido > 30.minutes) { /* ... */ }
```

`Instant`, `Clock` y `Duration` vienen de la librería estándar de Kotlin (`kotlin.time`). En versiones recientes, `Instant` y `Clock` se movieron de kotlinx-datetime a la librería estándar; según tu versión de Kotlin pueden requerir `@OptIn(ExperimentalTime::class)` o importarse desde `kotlinx.datetime`. kotlinx-datetime aporta lo de **calendario**: `LocalDate`, `LocalDateTime`, `TimeZone` y las conversiones.

## Inyecta el `Clock`

Si el código llama a `Clock.System.now()` directamente, **no puedes testearlo**: ¿cómo pruebas que un token vence a los 15 minutos sin esperar 15 minutos?

```kotlin
class EmitirToken(
  private val clock: Clock,          // el reloj es una dependencia más
) {
  fun ejecutar(usuario: UsuarioId): Token {
    val ahora = clock.now()
    return Token(usuario, emitidoEn = ahora, venceEn = ahora + 15.minutes)
  }
}
```

En producción, Koin le pasa `Clock.System`. En los tests, un reloj que **tú** controlas:

```kotlin
class RelojFijo(var ahora: Instant) : Clock {
  override fun now(): Instant = ahora
}

test("el token vence a los 15 minutos") {
  val reloj = RelojFijo(Instant.parse("2026-10-12T10:00:00Z"))
  val token = EmitirToken(reloj).ejecutar(UsuarioId("u-1"))

  token.venceEn shouldBe Instant.parse("2026-10-12T10:15:00Z")
}
```

Con `reloj.ahora += 16.minutes` pruebas el caso "ya venció" al instante. Es la misma idea que el [Kotest](../19-testing/01-kotest.md) con fakes: el tiempo es otro puerto.

## Convertir a hora local: solo cuando hace falta

Hay reglas de negocio que **sí** dependen de la zona: "el reporte diario incluye los pedidos de **hoy**", pero hoy en Buenos Aires empieza 3 horas después que en UTC.

```kotlin
val zona = TimeZone.of("America/Argentina/Buenos_Aires")

// De instante a calendario
val local: LocalDateTime = pedido.creadoEn.toLocalDateTime(zona)
val dia: LocalDate = local.date

// De calendario a instantes: los límites de "hoy" en esa zona
val hoy: LocalDate = clock.now().toLocalDateTime(zona).date
val inicio: Instant = hoy.atStartOfDayIn(zona)
val fin: Instant = hoy.plus(1, DateTimeUnit.DAY).atStartOfDayIn(zona)

// Consulta: creado_en >= inicio AND creado_en < fin
```

Fíjate en el patrón: la base sigue trabajando con instantes; la zona se usa **solo** para calcular los límites.

**¿De dónde sale la zona?** Nunca de la zona del servidor (`TimeZone.currentSystemDefault()`): el servidor puede estar en cualquier lado y en producción suele estar en UTC. La zona es un **dato de negocio**: la configuración de la empresa o la preferencia del usuario. Guárdala con su identificador IANA (`"America/Argentina/Buenos_Aires"`), nunca como un desplazamiento fijo (`-03:00`), porque los desplazamientos cambian con el horario de verano.

## Trampas del calendario

* **"Sumar un día" no siempre es sumar 24 horas.** En un cambio de horario de verano, un día local puede durar 23 o 25 horas. Para "mañana a la misma hora", suma un día de **calendario** (`DateTimeUnit.DAY` con la zona), no `24.hours`.
* **Hay horas locales que no existen o se repiten.** Cuando el reloj se adelanta, las 02:30 pueden no existir; cuando se atrasa, pueden ocurrir dos veces. Las funciones de conversión resuelven estos casos con reglas definidas: no las implementes a mano.
* **Programar "todos los lunes a las 08:00 hora local"** se calcula con la zona en cada ejecución, no sumando 7 días en UTC, o se corre una hora dos veces al año.

## Cómo se guarda y cómo viaja

| Dónde | Cómo |
| --- | --- |
| PostgreSQL | `timestamptz` (guarda el instante en UTC), como viste en [PostgreSQL y HikariCP](../15-persistencia/03-postgresql-y-hikaricp.md) |
| Exposed | Columna `timestamp(...)` del módulo de fechas, que se mapea a `Instant` |
| JSON | ISO 8601 en UTC: `"2026-10-12T13:00:00Z"` |
| Una fecha sin hora | `date` en Postgres, `LocalDate` en Kotlin, `"2026-10-12"` en JSON |

kotlinx.serialization ya sabe serializar `Instant` y `LocalDate` como texto ISO 8601. Para leer o escribir a mano:

```kotlin
val instante = Instant.parse("2026-10-12T13:00:00Z")
val fecha = LocalDate.parse("2026-10-12")
```

Si un proveedor manda epoch, **lee la documentación**: `1760274000` en segundos y `1760274000000` en milisegundos son el mismo instante, y confundirlos te lleva al año 1970 o al 57.000. Lo viste en [Ktor Client y APIs de terceros](../17-resiliencia-e-integraciones/03-ktor-client-y-apis-de-terceros.md).

Para interoperar con librerías Java (JWT, JDBC), existen conversiones como `toJavaInstant()`.

## Medir duraciones: otro reloj

Para medir cuánto tarda algo, **no** restes dos `Clock.System.now()`: el reloj del sistema puede ajustarse (por sincronización NTP) y darte duraciones negativas. Usa el reloj monótono:

```kotlin
val marca = TimeSource.Monotonic.markNow()
procesarLote()
val duracion: Duration = marca.elapsedNow()
```

`Clock` es para "¿qué hora es?"; `TimeSource.Monotonic` es para "¿cuánto tardó?". Lo usaste en el plugin de [Ktor server y plugins](../14-ktor-y-koin/01-ktor-server-y-plugins.md).

-----

## Resumen en 5 líneas

1. Un `Instant` es un momento universal; `LocalDate` y `LocalDateTime` son de calendario y necesitan una zona para ser un momento.
2. El backend guarda y transmite instantes: `timestamptz` en Postgres, ISO 8601 en UTC en el JSON.
3. Inyecta el `Clock` como dependencia: en los tests, un reloj fijo que tú controlas.
4. Convierte a hora local solo para reglas de negocio, con la zona IANA de la empresa o del usuario, nunca la del servidor.
5. Un día local no siempre dura 24 horas; para medir duraciones usa `TimeSource.Monotonic`, no el reloj de pared.

-----

## Siguiente lección

Terminaste serialización y tiempo. Lo próximo es [Arquitectura](../12-arquitectura/README.md). Vuelve al índice: [README principal](../README.md).
