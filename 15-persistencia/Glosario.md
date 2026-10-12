# Glosario

Las palabras que aparecen en las lecciones de esta carpeta, explicadas de forma simple. Están en orden alfabético.

-----

**ACID.** Las cuatro garantías de una transacción: atomicidad, consistencia, aislamiento y durabilidad.

**Actualización perdida (lost update).** Dos transacciones leen el mismo valor, ambas lo modifican y la segunda pisa a la primera.

**Advisory lock.** Lock con nombre de Postgres que no depende de una tabla. Está atado a la conexión, lo que lo vuelve delicado con un pool.

**Anomalía.** Un resultado incorrecto causado por transacciones concurrentes.

**Bloqueo optimista.** No bloquear; al escribir, verificar que nadie cambió el dato mientras tanto (por ejemplo, con una columna `version`).

**Bloqueo pesimista.** Bloquear la fila al leerla (`SELECT ... FOR UPDATE`), asumiendo que alguien más la va a querer modificar.

**Checksum.** Huella del contenido de un archivo; si el archivo cambia, el checksum cambia.

**Clave foránea (FK).** Columna que apunta a la clave primaria de otra tabla.

**Clave primaria (PK).** Columna (o columnas) que identifica cada fila de forma única.

**Compare-and-swap (CAS).** Escribir un valor nuevo solo si el valor actual es el que esperabas.

**Conexión.** Un canal abierto entre tu app y la base. Abrirla cuesta red, autenticación y memoria en el servidor.

**Constraint.** Regla que la base hace cumplir al insertar o actualizar (`NOT NULL`, `UNIQUE`, `CHECK`, `FOREIGN KEY`).

**DAO de Exposed.** Estilo con clases entidad que se cargan y se guardan como objetos.

**`DataSource`.** La interfaz de JDBC que entrega conexiones; HikariCP la implementa.

**Deadlock.** Dos transacciones esperándose mutuamente para siempre. Postgres lo detecta y aborta una.

**DSL de Exposed.** Estilo de Exposed que se parece a SQL: `selectAll().where { ... }`.

**Expand/contract.** Técnica para cambiar el esquema en pasos compatibles con la versión anterior de la app, sin cortar el servicio.

**Exposed.** Librería de acceso a bases de datos para Kotlin, de JetBrains.

**Flyway.** Herramienta que aplica migraciones en orden y lleva el registro en `flyway_schema_history`.

**HikariCP.** Pool de conexiones JDBC rápido y liviano.

**Índice.** Estructura auxiliar (normalmente un B-tree) que acelera las búsquedas por ciertas columnas.

**JDBC.** La API estándar de la JVM para hablar con bases de datos. Es bloqueante.

**Lease.** Un lock con fecha de vencimiento guardado en una tabla: "esto es mío hasta tal hora".

**Lectura fantasma (phantom read).** Una consulta repetida dentro de la misma transacción devuelve filas nuevas que otra insertó.

**Lectura no repetible.** El mismo dato, leído dos veces en la misma transacción, devuelve valores distintos.

**Lectura sucia (dirty read).** Leer un cambio que otra transacción todavía no confirmó. Postgres nunca la permite.

**Migración.** Archivo con los cambios de esquema de un paso (crear tabla, agregar columna).

**Nivel de aislamiento.** Cuánto se protege una transacción de los cambios de las demás (`READ COMMITTED`, `REPEATABLE READ`, `SERIALIZABLE`).

**Normalización.** Organizar las tablas para que cada dato se guarde en un solo lugar.

**Pool de conexiones.** Conjunto de conexiones abiertas que se reutilizan.

**Problema N+1.** Hacer una consulta para traer N elementos y después una consulta más por cada uno.

**`ResultRow`.** Una fila del resultado de una consulta de Exposed.

**Schema (Postgres).** Un espacio de nombres dentro de una base: agrupa tablas, como una carpeta.

**`SKIP LOCKED`.** Variante de `FOR UPDATE` que saltea las filas bloqueadas; ideal para colas de trabajo.

**Snapshot.** Una "foto" de la base en un momento dado, que la transacción usa para leer.

**Tabla.** Conjunto de filas con las mismas columnas.

**`timestamptz`.** Tipo de Postgres que guarda un instante en UTC. Se mapea a `Instant` en Kotlin.

**Transacción.** Grupo de operaciones que se confirman juntas (`COMMIT`) o se deshacen juntas (`ROLLBACK`).

**Write skew.** Dos transacciones leen lo mismo, deciden algo válido por separado y escriben filas distintas, rompiendo una regla entre las dos.
