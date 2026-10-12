# Glosario

Las palabras que aparecen en las lecciones de esta carpeta, explicadas de forma simple. Están en orden alfabético.

-----

**`Clock`.** La fuente de la hora actual. Se inyecta como dependencia para poder controlarla en los tests.

**Deserializar.** Convertir de un formato (JSON) a un objeto.

**Discriminador.** El campo del JSON que dice qué subtipo concreto es un objeto polimórfico, normalmente `"type"`.

**`Duration`.** Una cantidad de tiempo: 15 minutos, 2 horas.

**Epoch.** Cantidad de segundos (o milisegundos) desde el 1 de enero de 1970 en UTC.

**Horario de verano (DST).** Cambio estacional de la hora local en algunas regiones.

**`Instant`.** Un punto exacto en el tiempo, independiente de cualquier zona horaria.

**ISO 8601.** Formato estándar para fechas y horas: `2026-10-12T13:00:00Z`.

**`Json`.** La clase de kotlinx.serialization que configura y ejecuta la conversión a JSON.

**`JsonElement`.** Representación de JSON sin forma fija, para leer datos cuya estructura no se conoce.

**`LocalDate` / `LocalDateTime`.** Una fecha (o fecha y hora) de calendario, sin zona horaria.

**Plugin del compilador.** Extensión del compilador de Kotlin que genera código durante la compilación.

**Polimorfismo (en JSON).** Guardar en el JSON qué subtipo concreto es un objeto, para reconstruirlo bien.

**Reloj monótono.** Reloj que solo avanza y no se ajusta; sirve para medir duraciones (`TimeSource.Monotonic`).

**`@SerialName`.** Anotación que define el nombre de un campo o subtipo en el JSON.

**Serializador.** El código que sabe convertir un tipo concreto a un formato y de vuelta.

**Serializar.** Convertir un objeto a un formato de texto o bytes, como JSON.

**UTC.** El tiempo universal coordinado; la referencia contra la que se definen todas las zonas.

**Zona horaria.** Las reglas de una región para pasar de UTC a la hora local, incluidos los cambios de horario de verano. Se identifica con un nombre IANA, como `America/Argentina/Buenos_Aires`.
