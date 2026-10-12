# Idempotencia

## En una frase

Una operación es idempotente si ejecutarla **una vez o diez veces deja el sistema en el mismo estado**; es la condición para poder reintentar sin miedo, porque en sistemas distribuidos todo mensaje puede llegar más de una vez.

-----

## Antes de empezar

Conviene que ya sepas:

* Por qué se reintenta: [Reintentos con backoff](01-reintentos-con-backoff.md).
* Constraints `UNIQUE` y compare-and-swap: [SQL relacional](../15-persistencia/01-sql-relacional.md) y [Bloqueos y concurrencia](../15-persistencia/06-bloqueos-y-concurrencia.md).

Palabras nuevas (también están en el [Glosario](Glosario.md)):

* **Idempotente:** Que repetirla no cambia el resultado más allá de la primera vez.
* **Idempotency key:** Identificador único que el cliente manda con una petición para que el servidor reconozca los duplicados.
* **Upsert:** Insertar si no existe, actualizar si existe, en una sola operación.
* **Al menos una vez (at-least-once):** Garantía de que algo ocurre una o más veces; puede haber duplicados.
* **Como mucho una vez (at-most-once):** Garantía de que algo ocurre cero o una vez; puede perderse.

-----

## El problema: el timeout ambiguo

```
Cliente ──POST /pagos──> Servidor
                         procesa el pago ✅
                         responde 201...
Cliente <──── ✂️ la conexión se corta ────
Cliente: "timeout, ¿se hizo o no?"
```

El cliente **no puede saber** si el pago se procesó. Si no reintenta, puede perder un pago. Si reintenta, puede cobrar dos veces. No hay forma de distinguirlo desde afuera.

La única salida: que **reintentar sea seguro**. Es decir, que la operación sea idempotente.

## Qué es idempotente por naturaleza

| Operación | ¿Idempotente? | Por qué |
| --- | --- | --- |
| `GET /pedidos/p1` | ✅ | Solo lee |
| `PUT /pedidos/p1` con el estado completo | ✅ | "Dejarlo así" es igual una o diez veces |
| `DELETE /pedidos/p1` | ✅ | Después de la primera, ya no existe |
| `POST /pedidos` | ❌ | Cada llamada crea uno nuevo |
| `stock = 5` | ✅ | Asignar un valor |
| `stock = stock - 1` | ❌ | Cada ejecución resta otra vez |

La regla general: **"dejar en este estado"** es idempotente; **"hacer un cambio relativo"** no lo es.

## Idempotency key: hacer idempotente un `POST`

El cliente genera un identificador único **por operación** (no por intento) y lo manda en cada intento:

```
POST /api/v1/pagos
Idempotency-Key: 7f3c9a2e-...
{ "pedidoId": "p1", "monto": 1500 }
```

El servidor guarda la clave junto con el resultado:

```sql
CREATE TABLE claves_idempotencia (
  clave        text NOT NULL,
  empresa_id   uuid NOT NULL,
  hash_peticion bytea NOT NULL,
  estado       text NOT NULL,      -- EN_CURSO | COMPLETADA
  respuesta    jsonb,
  creada_en    timestamptz NOT NULL DEFAULT now(),
  PRIMARY KEY (empresa_id, clave)
);
```

El flujo:

1. **Llega una clave nueva:** se inserta como `EN_CURSO`, se procesa, se guarda la respuesta y se marca `COMPLETADA`.
2. **Llega una clave ya completada:** se devuelve la respuesta guardada, **sin** procesar de nuevo.
3. **Llega una clave en curso** (dos intentos simultáneos): la `PRIMARY KEY` impide la segunda inserción, y se responde `409 Conflict` para que el cliente reintente después.
4. **Llega la misma clave con otro cuerpo:** el `hash_peticion` no coincide. Es un error del cliente: `422`.

La clave se guarda por un tiempo limitado (por ejemplo, 24 horas), lo suficiente para cubrir los reintentos.

Fíjate en que la clave es **por empresa**: si fuera global, dos clientes distintos que por casualidad usen la misma clave se pisarían.

## Idempotencia en sincronizaciones

Un job que trae datos de un proveedor cada minuto **va a recibir los mismos datos muchas veces**. Si cada ejecución hace `INSERT`, la tabla se llena de duplicados.

La solución es identificar cada registro por **su ID en el sistema externo** y hacer un **upsert**:

```sql
CREATE TABLE dispositivos (
  id            uuid PRIMARY KEY,
  proveedor     text NOT NULL,
  id_externo    text NOT NULL,
  bateria       int,
  actualizado_en timestamptz NOT NULL,
  UNIQUE (proveedor, id_externo)
);

INSERT INTO dispositivos (id, proveedor, id_externo, bateria, actualizado_en)
VALUES (gen_random_uuid(), 'proveedor-a', 'R-001', 87, now())
ON CONFLICT (proveedor, id_externo)
DO UPDATE SET bateria = EXCLUDED.bateria, actualizado_en = EXCLUDED.actualizado_en;
```

Correr esto una o cien veces deja la misma fila. La sincronización completa se vuelve idempotente.

En Exposed existe `upsert` para esto mismo.

**Cuidado con el orden:** si dos ejecuciones procesan datos de distintos momentos, la más vieja puede pisar a la más nueva. Agrega una condición: actualizar solo si el dato que llega es más reciente.

```sql
DO UPDATE SET ...
WHERE dispositivos.actualizado_en < EXCLUDED.actualizado_en;
```

## Eventos y notificaciones: deduplicar

Si procesas eventos (webhooks de un proveedor, mensajes de una cola), lo normal es que la entrega sea **al menos una vez**: puede llegar el mismo evento dos veces. Guarda los IDs ya procesados:

```kotlin
// En la misma transacción que el efecto del evento
val esNuevo = eventosProcesados.registrar(evento.id)  // INSERT con UNIQUE; false si ya existía
if (!esNuevo) return                                  // duplicado: ignorar
aplicarEfecto(evento)
```

Registrar el evento y aplicar el efecto **en la misma transacción** es clave: si el efecto falla, el registro también se deshace, y el próximo intento lo procesa.

## Lo que no es transaccional: el mundo exterior

Un email, una llamada a otra API o un SMS **no** se deshacen con un `ROLLBACK`. Ahí tienes que elegir:

| Estrategia | Si algo falla a mitad | Riesgo |
| --- | --- | --- |
| Registrar → enviar | Queda registrado pero no enviado | Se pierde el envío (como mucho una vez) |
| Enviar → registrar | Se envió pero no quedó registrado | Se envía dos veces (al menos una vez) |

No hay opción perfecta. Para una alerta, suele ser mejor un duplicado que perderla. Para un cobro, al revés. Es una decisión de negocio, no técnica. Lo verás aplicado en [Correo SMTP](06-correo-smtp.md).

El patrón **outbox** lo hace más robusto: en la misma transacción que el cambio de negocio, se inserta un mensaje en una tabla `outbox`; un job lo envía después y lo marca como enviado. Así nunca se pierde un mensaje por un cambio que sí se confirmó.

-----

## Resumen en 5 líneas

1. Idempotente es que repetir la operación no cambie el resultado: es lo que vuelve seguro reintentar.
2. `GET`, `PUT` y `DELETE` lo son por naturaleza; un `POST` necesita una idempotency key guardada con su respuesta.
3. Las sincronizaciones se vuelven idempotentes con un upsert por ID externo, y una condición para no pisar datos más nuevos.
4. Los eventos se deduplican registrando su ID en la misma transacción que su efecto.
5. Lo externo (emails, APIs) no tiene rollback: elige entre al menos una vez y como mucho una vez, o usa un outbox.

-----

## Siguiente lección

[3. Ktor Client y APIs de terceros](03-ktor-client-y-apis-de-terceros.md): la integración completa.
