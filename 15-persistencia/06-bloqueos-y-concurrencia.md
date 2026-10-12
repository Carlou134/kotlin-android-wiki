# Bloqueos y concurrencia: pesimista, optimista y leases

## En una frase

Cuando dos procesos quieren modificar el mismo dato, o **esperan su turno** (bloqueo pesimista), o **trabajan sin esperar y detectan el conflicto al escribir** (bloqueo optimista); un lease extiende la idea a "esta tarea es mía por un rato".

-----

## Antes de empezar

Conviene que ya sepas:

* Qué es una actualización perdida: [Transacciones y aislamiento](02-transacciones-y-aislamiento.md).
* Que `update` de Exposed devuelve las filas afectadas: [Exposed DSL](04-exposed-dsl.md).
* El problema de los jobs en varias instancias: [Jobs programados](../10-corrutinas/04-jobs-programados.md).
* Outcomes con `sealed interface`: [La capa application](../12-arquitectura/05-capa-application.md).

Palabras nuevas (también están en el [Glosario](Glosario.md)):

* **Bloqueo pesimista:** Bloquear la fila al leerla, asumiendo que alguien más la va a querer modificar.
* **Bloqueo optimista:** No bloquear; al escribir, verificar que nadie cambió el dato mientras tanto.
* **Compare-and-swap (CAS):** Escribir un valor nuevo solo si el valor actual es el que esperabas.
* **Lease:** Un lock con fecha de vencimiento: "esto es mío hasta tal hora".
* **Deadlock:** Dos transacciones esperándose mutuamente para siempre.

-----

## El problema, otra vez

```
T1: lee el pedido (estado = PENDIENTE)
T2: lee el pedido (estado = PENDIENTE)
T1: lo marca como ENVIADO, COMMIT
T2: lo marca como CANCELADO, COMMIT     ❌ se canceló un pedido ya enviado
```

Cada transacción, por separado, tomó una decisión válida con lo que leyó. El problema es que lo que leyó **cambió** antes de escribir.

## Bloqueo pesimista: `SELECT ... FOR UPDATE`

"Asumo que va a haber conflicto, así que bloqueo antes de decidir."

```sql
BEGIN;
SELECT * FROM pedidos WHERE id = 'p1' FOR UPDATE;   -- bloquea la fila
-- decido con el dato bloqueado
UPDATE pedidos SET estado = 'CANCELADO' WHERE id = 'p1';
COMMIT;                                             -- libera el bloqueo
```

Si T2 intenta el mismo `SELECT ... FOR UPDATE`, **espera** hasta que T1 confirme, y después lee el dato ya actualizado (`ENVIADO`). Entonces decide bien.

En Exposed:

```kotlin
transaction {
  val pedido = PedidosTable
    .selectAll()
    .where { PedidosTable.id eq pedidoId }
    .forUpdate()
    .single()
    .toPedido()
  // decidir y actualizar dentro de la misma transacción
}
```

Variantes útiles de Postgres:

| Variante | Comportamiento |
| --- | --- |
| `FOR UPDATE` | Espera a que se libere |
| `FOR UPDATE NOWAIT` | Si está bloqueada, falla en el acto |
| `FOR UPDATE SKIP LOCKED` | Saltea las filas bloqueadas y sigue con las otras |

`SKIP LOCKED` es ideal para **colas de trabajo en la base**: varias instancias toman tareas pendientes sin pisarse, porque cada una saltea las que otra ya tomó.

```sql
SELECT * FROM tareas WHERE estado = 'PENDIENTE'
ORDER BY creada_en
LIMIT 10
FOR UPDATE SKIP LOCKED;
```

**Costos:** la transacción retiene el bloqueo hasta el `COMMIT`, así que tiene que ser corta. Y si T1 bloquea A y después B, mientras T2 bloquea B y después A, se produce un **deadlock**: Postgres lo detecta y aborta una de las dos. Bloquear siempre en el mismo orden lo evita.

## Bloqueo optimista: columna `version`

"Asumo que no va a haber conflicto; si lo hay, lo detecto al escribir."

La tabla tiene una columna `version`. Cada escritura exige que la versión siga siendo la que leíste, y la incrementa:

```sql
UPDATE pedidos
SET estado = 'CANCELADO', version = version + 1
WHERE id = 'p1' AND version = 3;
```

* Si nadie lo tocó: 1 fila afectada. Éxito, ahora es la versión 4.
* Si alguien lo cambió: la versión ya no es 3, así que hay **0 filas afectadas**. Conflicto detectado.

En el repositorio, ese número se convierte en un resultado explícito:

```kotlin
sealed interface GuardarResultado {
  data object Guardado : GuardarResultado
  data object ConflictoDeVersion : GuardarResultado
}

override suspend fun actualizar(pedido: Pedido): GuardarResultado = dbQuery {
  val filas = PedidosTable.update({
    (PedidosTable.id eq pedido.id.value) and (PedidosTable.version eq pedido.version)
  }) {
    it[estado] = pedido.estado.name
    it[version] = pedido.version + 1
  }
  if (filas == 1) GuardarResultado.Guardado else GuardarResultado.ConflictoDeVersion
}
```

Y el use case decide qué hacer: reintentar leyendo de nuevo, o devolver un outcome `ModificadoPorOtro` que la ruta traduce a `409 Conflict`.

## Compare-and-swap: la idea general

El versionado optimista es un caso de **compare-and-swap**: "cambia el valor solo si sigue siendo el que espero". No hace falta una columna `version`; se puede comparar cualquier campo.

Ejemplo clásico: un token de un solo uso (como un refresh token).

```sql
UPDATE tokens
SET usado_en = now()
WHERE id = 't1' AND usado_en IS NULL;
```

* 1 fila afectada: eras el primero, el token es válido.
* 0 filas afectadas: alguien ya lo usó. Si llegan dos peticiones con el mismo token al mismo tiempo, **solo una** gana, sin bloqueos y sin carreras. Esto se usa para detectar la reutilización de tokens en `16-seguridad`.

## ¿Pesimista u optimista?

| | Pesimista (`FOR UPDATE`) | Optimista (versión / CAS) |
| --- | --- | --- |
| Conflictos | Frecuentes | Raros |
| Qué pasa ante un conflicto | El segundo espera | El segundo falla y decide (reintentar u outcome) |
| Duración | Transacción corta, todo en una | Puede abarcar varias peticiones (leo, el usuario edita, guardo) |
| Riesgo | Deadlocks, conexiones retenidas | Reintentos si hay mucha contención |
| Ejemplo | Cola de tareas con `SKIP LOCKED` | Edición de un formulario, tokens de un solo uso |

La pregunta clave: ¿puedo tener la transacción abierta mientras decido? Si la decisión depende de un humano o de una API externa, **no**: usa optimista.

## Leases: "esto es mío por un rato"

Volvamos al problema de los [jobs programados](../10-corrutinas/04-jobs-programados.md): 3 instancias, y la sincronización solo debe correr en una.

Un `FOR UPDATE` no sirve, porque la sincronización tarda minutos y no puedes tener una transacción abierta todo ese tiempo. La solución es un **lease**: una fila que dice quién tiene la tarea y hasta cuándo.

```sql
CREATE TABLE leases (
  nombre     text PRIMARY KEY,
  dueno      text NOT NULL,
  expira_en  timestamptz NOT NULL
);
```

Tomar el lease es un compare-and-swap atómico:

```sql
INSERT INTO leases (nombre, dueno, expira_en)
VALUES ('sincronizacion', 'instancia-2', now() + interval '2 minutes')
ON CONFLICT (nombre) DO UPDATE
  SET dueno = EXCLUDED.dueno, expira_en = EXCLUDED.expira_en
  WHERE leases.expira_en < now();      -- solo si el anterior venció
```

* Si no existía o el anterior venció: 1 fila afectada, el lease es tuyo.
* Si otra instancia lo tiene vigente: 0 filas, no trabajas esta vez.

El job queda así:

```kotlin
lanzarJobPeriodico("sincronizacion", 1.minutes) {
  if (leases.intentarTomar("sincronizacion", instanciaId, duracion = 2.minutes)) {
    try {
      sincronizar.ejecutar()
    } finally {
      withContext(NonCancellable) { leases.liberar("sincronizacion", instanciaId) }
    }
  }
}
```

**¿Por qué vence?** Si la instancia que tiene el lease se cae a mitad del trabajo, nunca lo libera. El vencimiento garantiza que otra lo pueda tomar después. Sin vencimiento sería un lock eterno.

**La trampa del lease:** si el trabajo tarda **más** que el lease (una pausa larga de la JVM, una API lenta), el lease vence, otra instancia lo toma y por un momento trabajan **dos**. Defensas:

* Dura más que el trabajo esperado, y **renuévalo** durante trabajos largos (otro CAS que exige `dueno = yo`).
* Haz que el trabajo sea **idempotente**, para que correr dos veces no cause daño ([Idempotencia](../17-resiliencia-e-integraciones/02-idempotencia.md)).

**Alternativa en Postgres:** los *advisory locks* (`pg_try_advisory_lock`) son locks con nombre que no dependen de una tabla. Pero están atados a la **conexión**, y con un pool de conexiones eso es delicado: si se devuelve la conexión al pool sin liberarlo, el lock queda tomado. El lease en una tabla es más explícito y más fácil de inspeccionar.

-----

## Resumen en 5 líneas

1. El bloqueo pesimista (`SELECT ... FOR UPDATE`) hace esperar al segundo; requiere transacciones cortas y un orden fijo para evitar deadlocks.
2. `SKIP LOCKED` permite que varias instancias consuman una cola en la base sin pisarse.
3. El bloqueo optimista escribe con `WHERE version = ?`; 0 filas afectadas significa conflicto, y se convierte en un resultado explícito.
4. Compare-and-swap es la idea general: actualizar solo si el valor sigue siendo el esperado; sirve para tokens de un solo uso.
5. Un lease es un lock con vencimiento guardado en una tabla, ideal para jobs en varias instancias; se renueva y se combina con trabajo idempotente.

-----

## Siguiente lección

Terminaste la persistencia. Vuelve al índice: [README principal](../README.md).
