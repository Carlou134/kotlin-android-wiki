# Migraciones con Flyway

## En una frase

Una migración es un archivo SQL numerado que cambia el esquema de la base; Flyway las aplica en orden y recuerda cuáles ya corrió, así todas las bases (tu máquina, staging, producción) tienen exactamente el mismo esquema.

-----

## Antes de empezar

Conviene que ya sepas:

* Crear tablas, índices y constraints: [SQL relacional](01-sql-relacional.md).

Palabras nuevas (también están en el [Glosario](Glosario.md)):

* **Migración:** Archivo con los cambios de esquema de un paso (crear tabla, agregar columna).
* **Flyway:** Herramienta que aplica migraciones en orden y lleva el registro.
* **`flyway_schema_history`:** Tabla donde Flyway anota qué migraciones aplicó, cuándo y con qué checksum.
* **Checksum:** Huella del contenido de un archivo; si el archivo cambia, el checksum cambia.
* **Expand/contract:** Técnica para cambiar el esquema en pasos compatibles, sin cortar el servicio.

-----

## El problema: "en mi máquina funciona"

Sin migraciones, cada uno modifica su base a mano. Alguien agrega una columna en su local, se olvida de hacerlo en producción, y el deploy rompe. Nadie sabe qué versión del esquema tiene cada base.

## Cómo funciona Flyway

Los archivos viven en `src/main/resources/db/migration/` y siguen una convención de nombre:

```
V001__crear_clientes.sql
V002__crear_pedidos.sql
V003__agregar_indice_pedidos_cliente.sql
```

* `V` + versión + **dos** guiones bajos + descripción.
* Se aplican en orden de versión.

Al arrancar, Flyway:

1. Lee `flyway_schema_history` para saber qué versiones ya aplicó.
2. Aplica las que faltan, en orden, cada una en su transacción (en Postgres, el DDL es transaccional: si falla a mitad, se deshace).
3. Registra cada una con su checksum.

## Correrlo al arrancar la app

```kotlin
fun migrar(dataSource: DataSource) {
  Flyway.configure()
    .dataSource(dataSource)
    .locations("classpath:db/migration")
    .load()
    .migrate()
}
```

Se llama en el composition root, **antes** de aceptar peticiones. Con varios schemas, cada uno puede tener su carpeta y su configuración (`.schemas("auth")` y `.locations("classpath:db/migration/auth")`).

> Desde Flyway 10, el soporte de cada base es un artefacto aparte: para Postgres necesitas también `flyway-database-postgresql`.

## La regla de oro: nunca edites una migración aplicada

Si `V002` ya corrió en producción y la modificas, su checksum cambia y Flyway **se niega a arrancar** (error de validación). Y aunque lo fuerces, producción ya tiene la versión vieja aplicada: tu cambio nunca llegaría.

¿Te equivocaste en `V002`? Escribe `V009__corregir_...sql`. Las migraciones son un **historial**, no un estado.

## Cambiar el esquema sin cortar el servicio

Durante un deploy, por unos minutos conviven la versión vieja y la nueva de la app contra la **misma** base. Una migración que rompe a la versión vieja corta el servicio.

Ejemplo peligroso: renombrar una columna.

```sql
ALTER TABLE pedidos RENAME COLUMN total TO importe; -- ❌ la app vieja sigue buscando "total"
```

La técnica **expand/contract** lo hace en pasos compatibles:

| Paso | Migración | Código |
| --- | --- | --- |
| 1. Expandir | Agregar `importe` (nullable) | Escribe en las dos columnas |
| 2. Rellenar | `UPDATE pedidos SET importe = total WHERE importe IS NULL` | Lee de `importe` |
| 3. Contraer | Hacer `importe` `NOT NULL` y borrar `total` | Ya no menciona `total` |

Cada paso es un deploy separado, y en ningún momento hay una versión de la app que no funcione con el esquema.

Otros cambios que requieren cuidado:

* **Agregar una columna `NOT NULL`** sin `DEFAULT` falla si la tabla tiene filas. Agrégala nullable, rellénala y después ponle el `NOT NULL`.
* **Crear un índice** sobre una tabla grande bloquea las escrituras. En Postgres existe `CREATE INDEX CONCURRENTLY`, que no bloquea, pero no puede correr dentro de una transacción (Flyway permite desactivar la transacción para esa migración).

## Migraciones y tests

Los tests de integración deben correr **las mismas migraciones** contra un Postgres real. Así, si una migración tiene un error, falla el test y no el deploy.

-----

## Resumen en 5 líneas

1. Una migración es un SQL versionado (`V001__descripcion.sql`); Flyway las aplica en orden y las registra en `flyway_schema_history`.
2. Se corre al arrancar, desde el composition root, antes de aceptar peticiones.
3. Nunca edites una migración ya aplicada: el checksum cambia y Flyway se niega a arrancar. Corrige con una nueva.
4. Durante un deploy conviven dos versiones de la app; los cambios destructivos se hacen con expand/contract.
5. Cuidado con `NOT NULL` sin default y con índices sobre tablas grandes; prueba las migraciones en los tests de integración.

-----

## Siguiente lección

[6. Bloqueos y concurrencia](06-bloqueos-y-concurrencia.md): cómo evitar que dos procesos se pisen.
