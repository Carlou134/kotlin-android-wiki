# Transacciones y niveles de aislamiento

## En una frase

Una transacción agrupa varias operaciones para que ocurran **todas o ninguna**, y su nivel de aislamiento define cuánto puede "ver" una transacción de lo que hacen las otras al mismo tiempo.

-----

## Antes de empezar

Conviene que ya sepas:

* Tablas y constraints: [SQL relacional](01-sql-relacional.md).

Palabras nuevas (también están en el [Glosario](Glosario.md)):

* **Transacción:** Grupo de operaciones que se confirman juntas (`COMMIT`) o se deshacen juntas (`ROLLBACK`).
* **ACID:** Las cuatro garantías de una transacción: atomicidad, consistencia, aislamiento y durabilidad.
* **Nivel de aislamiento:** Cuánto se protege una transacción de los cambios de las demás.
* **Anomalía:** Un resultado incorrecto causado por transacciones concurrentes.
* **Snapshot:** Una "foto" de la base en un momento dado, que la transacción usa para leer.

-----

## El problema: quedarse a mitad de camino

Transferir dinero son dos operaciones:

```sql
UPDATE cuentas SET saldo = saldo - 100 WHERE id = 'A';
UPDATE cuentas SET saldo = saldo + 100 WHERE id = 'B';
```

Si el servidor se cae entre las dos, desaparecieron 100 pesos. La transacción lo evita:

```sql
BEGIN;
UPDATE cuentas SET saldo = saldo - 100 WHERE id = 'A';
UPDATE cuentas SET saldo = saldo + 100 WHERE id = 'B';
COMMIT;   -- o ROLLBACK si algo falló
```

## ACID

| Letra | Garantía | Significa |
| --- | --- | --- |
| **A** | Atomicidad | Todo o nada |
| **C** | Consistencia | Al terminar, se cumplen todos los constraints |
| **I** | Aislamiento | Las transacciones concurrentes no se ven a medias |
| **D** | Durabilidad | Después del `COMMIT`, el dato sobrevive a un corte de luz |

La "I" es la interesante, porque **no es absoluta**: depende del nivel que elijas.

## Las anomalías

Imaginá dos transacciones, T1 y T2, corriendo al mismo tiempo.

**Lectura sucia (dirty read):** T1 lee un cambio de T2 que todavía no se confirmó. Si T2 hace `ROLLBACK`, T1 trabajó con un dato que nunca existió. Postgres **nunca** permite esto.

**Lectura no repetible (non-repeatable read):** T1 lee el saldo (100), T2 lo cambia y confirma, T1 lo vuelve a leer y ahora es 50. Dentro de la misma transacción, el mismo dato cambió.

**Lectura fantasma (phantom read):** T1 cuenta los pedidos pendientes (5), T2 inserta uno nuevo y confirma, T1 vuelve a contar y son 6.

**Actualización perdida (lost update):** la más peligrosa en código de aplicación.

```
stock = 10
T1: lee stock -> 10
T2: lee stock -> 10
T1: escribe stock = 10 - 1 = 9, COMMIT
T2: escribe stock = 10 - 1 = 9, COMMIT    ❌ se vendieron 2, el stock dice 9
```

**Write skew:** T1 y T2 leen el mismo conjunto, cada una toma una decisión válida por separado y escriben filas **distintas**. Por ejemplo: "tiene que quedar al menos un médico de guardia". Hay dos de guardia, los dos piden salir a la vez, cada transacción ve "queda el otro" y ambos salen.

## Niveles de aislamiento en PostgreSQL

| Nivel | Lectura no repetible | Fantasma | Lost update | Write skew |
| --- | --- | --- | --- | --- |
| `READ COMMITTED` (**default**) | Puede ocurrir | Puede ocurrir | Puede ocurrir | Puede ocurrir |
| `REPEATABLE READ` | No | No | Detectada: error | Puede ocurrir |
| `SERIALIZABLE` | No | No | Detectada: error | Detectada: error |

Detalles de Postgres que conviene saber:

* `READ UNCOMMITTED` existe en el estándar, pero Postgres lo trata igual que `READ COMMITTED`.
* En `READ COMMITTED`, **cada sentencia** ve un snapshot nuevo, con lo confirmado hasta ese momento.
* En `REPEATABLE READ`, **toda la transacción** ve el snapshot del inicio. Si intentas modificar una fila que otra transacción cambió y confirmó, Postgres lanza un error de serialización (`SQLSTATE 40001`).
* En `SERIALIZABLE`, Postgres garantiza que el resultado es igual a ejecutar las transacciones una detrás de la otra. Si detecta un conflicto, aborta una con el mismo error `40001`.

**Consecuencia práctica:** con `REPEATABLE READ` o `SERIALIZABLE`, tu código **tiene que reintentar** la transacción entera cuando recibe `40001`. No es un bug: es el contrato.

## Cómo resolver el lost update sin subir el nivel

La mayoría de las apps usan `READ COMMITTED` y resuelven los conflictos puntuales de otra forma.

**1. Que la base haga la cuenta (operación atómica):**

```sql
UPDATE productos SET stock = stock - 1 WHERE id = 'x' AND stock > 0;
```

No hay "leer y después escribir": la base lee y escribe en un solo paso. Si devuelve 0 filas afectadas, no había stock. Es la mejor opción **cuando la lógica entra en una sola sentencia**.

**2. Bloqueo pesimista (`SELECT ... FOR UPDATE`):** bloquear la fila al leerla.

**3. Bloqueo optimista (columna `version`):** detectar el conflicto al escribir.

Las opciones 2 y 3 se ven en detalle en la [lección 6](06-bloqueos-y-concurrencia.md).

## Transacciones cortas

Una transacción abierta mantiene locks y una conexión ocupada. Reglas prácticas:

* **Nunca** llames a una API externa dentro de una transacción. Si tarda 10 segundos, bloqueas la conexión y las filas 10 segundos.
* Primero junta los datos de afuera, después abre la transacción, escribe y confirma.
* Cuanto más corta la transacción, menos conflictos.

-----

## Resumen en 5 líneas

1. Una transacción es todo o nada: `COMMIT` confirma todo, `ROLLBACK` deshace todo.
2. ACID garantiza atomicidad, consistencia, aislamiento y durabilidad; el aislamiento depende del nivel elegido.
3. Postgres usa `READ COMMITTED` por defecto: no tiene lecturas sucias, pero sí puede perder actualizaciones.
4. `REPEATABLE READ` y `SERIALIZABLE` detectan conflictos con el error `40001`, y el código debe reintentar.
5. El lost update se evita con operaciones atómicas en SQL, bloqueo pesimista o bloqueo optimista; y las transacciones siempre cortas.

-----

## Siguiente lección

[3. PostgreSQL y HikariCP](03-postgresql-y-hikaricp.md): cómo conectarse a la base sin desperdiciar conexiones.
