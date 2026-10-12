# Serialización y tiempo

En esta carpeta aprendes dos cosas que aparecen en **cada** petición de un backend: cómo convertir objetos de Kotlin a JSON y de vuelta con **kotlinx.serialization**, y cómo manejar fechas y horas sin errores de zona horaria con **`Instant`** y **kotlinx.datetime**.

-----

## Antes de empezar

Conviene que ya hayas leído [POO avanzada](../08-poo-avanzada/README.md) (data classes, `sealed`, `value class` y `object`) y [Null safety y errores](../07-null-safety-y-errores/README.md).

Si en algún momento aparece una palabra que no conoces, búscala en el [Glosario](Glosario.md).

-----

## Orden de lectura

Las lecciones son independientes, pero conviene leerlas en este orden:

| Lección | Qué aprendes | Qué necesitas saber antes |
| --- | --- | --- |
| [1. kotlinx.serialization](01-kotlinx-serialization.md) | `@Serializable`, configuración de `Json`, `@SerialName`, opcionales, enums, `sealed` polimórficos y serializadores propios | Data classes y `sealed` |
| [2. Tiempo con `Instant` y kotlinx.datetime](02-tiempo-con-instant.md) | `Instant` vs fechas locales, zonas horarias, `Duration`, un `Clock` inyectable y cómo se guarda y viaja el tiempo | Serialización |

-----

## El mapa completo en una mirada

```
Serialización  ->  "objetos <-> JSON, verificado al compilar"
   @Serializable data class PedidoResponse(val id: String, @SerialName("created_at") val creadoEn: Instant)
   Json { ignoreUnknownKeys = true }.decodeFromString<PedidoResponse>(texto)
   sealed + "type": "aprobado"  ->  polimorfismo

Tiempo         ->  "un instante es universal; la hora local depende de dónde"
   val ahora: Instant = clock.now()               // siempre UTC por dentro
   ahora + 15.minutes                             // aritmética con Duration
   ahora.toLocalDateTime(TimeZone.of("America/Argentina/Buenos_Aires"))
   LocalDate(2026, 10, 12)                        // una fecha sin hora ni zona
```

-----

## Cómo está armada cada lección

1. **En una frase:** qué es, sin jerga.
2. **Antes de empezar:** qué tienes que saber y las palabras nuevas.
3. **El contenido:** la explicación con ejemplos.
4. **Resumen en 5 líneas:** para repasar.
5. **Siguiente lección:** para seguir el orden.

-----

## Recursos

* [Guía de kotlinx.serialization](https://kotlinlang.org/docs/serialization.html)
* [kotlinx-datetime en GitHub](https://github.com/Kotlin/kotlinx-datetime)

-----

## Después de esta carpeta

Ya sabes cómo viajan los datos y el tiempo. Lo próximo es [Arquitectura](../12-arquitectura/README.md). Vuelve al índice: [README principal](../README.md).
