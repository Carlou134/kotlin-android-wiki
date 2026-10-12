# Persistencia

En esta carpeta aprendes a **guardar datos sin perderlos ni corromperlos**: cómo modelar tablas en una base relacional, qué garantiza una transacción, cómo conectarse a PostgreSQL de forma eficiente, cómo escribir consultas con Exposed, cómo versionar el esquema con Flyway y cómo evitar que dos procesos se pisen al modificar el mismo dato.

-----

## Antes de empezar

Conviene que ya hayas leído [Arquitectura](../12-arquitectura/README.md), porque el repositorio es un adapter secundario, y [Corrutinas](../10-corrutinas/README.md), porque JDBC es bloqueante y hay que saber dónde correrlo.

Si en algún momento aparece una palabra que no conoces, búscala en el [Glosario](Glosario.md).

-----

## Orden de lectura

Las lecciones se apoyan una en la otra, así que conviene leerlas en este orden:

| Lección | Qué aprendes | Qué necesitas saber antes |
| --- | --- | --- |
| [1. SQL relacional](01-sql-relacional.md) | Tablas, claves, constraints, normalización e índices | Nada |
| [2. Transacciones y aislamiento](02-transacciones-y-aislamiento.md) | ACID, anomalías de concurrencia y niveles de aislamiento | SQL relacional |
| [3. PostgreSQL y HikariCP](03-postgresql-y-hikaricp.md) | Tipos útiles de Postgres, schemas y pool de conexiones | SQL relacional |
| [4. Exposed DSL](04-exposed-dsl.md) | Tablas, consultas, transacciones y mapeo a dominio dentro del repositorio | Arquitectura hexagonal y corrutinas |
| [5. Migraciones con Flyway](05-migraciones-con-flyway.md) | Versionar el esquema y cambiarlo sin cortar el servicio | SQL relacional |
| [6. Bloqueos y concurrencia](06-bloqueos-y-concurrencia.md) | `SELECT ... FOR UPDATE`, versionado optimista, compare-and-swap y leases | Transacciones |

-----

## El mapa completo en una mirada

```
SQL relacional         ->  "la forma de los datos"
   PRIMARY KEY, FOREIGN KEY, UNIQUE, CHECK, NOT NULL
   CREATE INDEX ... ON pedidos (cliente_id, creado_en)

Transacciones          ->  "todo o nada"
   BEGIN; ... COMMIT;  / ROLLBACK;
   READ COMMITTED (default de Postgres) < REPEATABLE READ < SERIALIZABLE

PostgreSQL + HikariCP  ->  "conexiones reutilizables"
   timestamptz, uuid, jsonb      schemas: public, auth
   HikariDataSource(maximumPoolSize = 10)

Exposed DSL            ->  "SQL tipado en Kotlin"
   object PedidosTable : Table("pedidos")
   PedidosTable.selectAll().where { PedidosTable.id eq id }.map { it.toPedido() }

Flyway                 ->  "el esquema también se versiona"
   db/migration/V001__crear_pedidos.sql   (nunca se edita una ya aplicada)

Bloqueos               ->  "que dos procesos no se pisen"
   pesimista: SELECT ... FOR UPDATE            (espero mi turno)
   optimista: UPDATE ... WHERE version = 3     (si cambió, conflicto)
   lease:     "esta tarea es mía hasta las 10:05"
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

* [Documentación de PostgreSQL: aislamiento de transacciones](https://www.postgresql.org/docs/current/transaction-iso.html)
* [Documentación de Exposed](https://www.jetbrains.com/help/exposed/home.html)
* [HikariCP: About Pool Sizing](https://github.com/brettwooldridge/HikariCP/wiki/About-Pool-Sizing)
* [Documentación de Flyway](https://documentation.red-gate.com/flyway)

-----

## Después de esta carpeta

Ya sabes guardar datos de forma segura. Vuelve al índice: [README principal](../README.md).
