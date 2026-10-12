# SQL y modelado relacional

## En una frase

Una base relacional guarda los datos en tablas conectadas por claves, y te deja declarar **reglas** (constraints) que la propia base hace cumplir, aunque tu código tenga un bug.

-----

## Antes de empezar

Conviene que ya sepas:

* Qué son las data classes, porque una fila se parece mucho a una: [Data classes](../06-poo/03-data-classes.md).

Palabras nuevas (también están en el [Glosario](Glosario.md)):

* **Tabla:** Conjunto de filas con las mismas columnas.
* **Clave primaria (PK):** Columna (o columnas) que identifica cada fila de forma única.
* **Clave foránea (FK):** Columna que apunta a la clave primaria de otra tabla.
* **Constraint:** Regla que la base hace cumplir al insertar o actualizar.
* **Normalización:** Organizar las tablas para que cada dato se guarde en un solo lugar.
* **Índice:** Estructura auxiliar que acelera las búsquedas por ciertas columnas.

-----

## Tablas y claves

```sql
CREATE TABLE clientes (
  id         uuid PRIMARY KEY,
  email      text NOT NULL UNIQUE,
  nombre     text NOT NULL
);

CREATE TABLE pedidos (
  id          uuid PRIMARY KEY,
  cliente_id  uuid NOT NULL REFERENCES clientes (id),
  total       numeric(12, 2) NOT NULL CHECK (total > 0),
  estado      text NOT NULL CHECK (estado IN ('PENDIENTE', 'ENVIADO', 'CANCELADO')),
  creado_en   timestamptz NOT NULL DEFAULT now()
);
```

* `PRIMARY KEY`: no puede haber dos pedidos con el mismo `id`, y no puede ser `NULL`.
* `REFERENCES clientes (id)`: es la clave foránea. No puedes crear un pedido para un cliente que no existe, ni borrar un cliente que tiene pedidos.

## Constraints: la última línea de defensa

| Constraint | Qué impide |
| --- | --- |
| `NOT NULL` | Que falte el dato |
| `UNIQUE` | Que se repita (dos clientes con el mismo email) |
| `CHECK` | Valores fuera de regla (total negativo, estado inventado) |
| `FOREIGN KEY` | Referencias a filas que no existen |

"Pero si ya valido en el dominio, ¿para qué repetir?" Porque el dominio no es el único que escribe:

* Dos peticiones simultáneas pueden pasar la validación "el email no existe" al mismo tiempo. Solo el `UNIQUE` evita el duplicado.
* Un script de migración, un bug o alguien con acceso a la base pueden saltarse tu código.

El dominio valida para dar **buenos mensajes de error**. La base valida para que los datos **nunca** queden inválidos. Son complementarias.

## Normalización: cada dato en un solo lugar

Una tabla sin normalizar:

| pedido_id | cliente_email | cliente_nombre | producto | precio |
| --- | --- | --- | --- | --- |
| p1 | ana@mail.com | Ana | Teclado | 50 |
| p2 | ana@mail.com | Ana | Mouse | 20 |

Si Ana cambia su nombre, hay que actualizar **todas** sus filas. Si te olvidas de una, los datos se contradicen. Eso se llama **anomalía de actualización**.

Las formas normales, en corto:

| Forma | Regla | En la práctica |
| --- | --- | --- |
| **1FN** | Cada celda tiene un solo valor | Nada de `"teclado,mouse"` en una columna |
| **2FN** | Cada columna depende de **toda** la clave primaria | Si la PK es `(pedido_id, producto_id)`, el nombre del cliente no va ahí |
| **3FN** | Las columnas no dependen de otras columnas que no son clave | El nombre del cliente depende del cliente, no del pedido: va en `clientes` |

Versión normalizada: `clientes`, `pedidos` (con `cliente_id`), `productos` y `lineas_pedido` (`pedido_id`, `producto_id`, `cantidad`, `precio_unitario`).

**Desnormalizar a propósito** es válido cuando lo necesitas. Por ejemplo, `precio_unitario` se copia en la línea del pedido porque el precio del producto cambia con el tiempo y el pedido tiene que recordar cuánto se cobró. Eso no es un dato repetido: es un dato **histórico**.

## Índices

Sin índice, buscar los pedidos de un cliente obliga a la base a leer **toda** la tabla (*sequential scan*). Con millones de filas, eso es lento.

```sql
CREATE INDEX idx_pedidos_cliente ON pedidos (cliente_id);
```

Un índice es como el índice de un libro: un árbol ordenado (B-tree) que lleva directo a las filas.

### Índices compuestos: el orden importa

```sql
CREATE INDEX idx_pedidos_cliente_fecha ON pedidos (cliente_id, creado_en);
```

Sirve para:

* `WHERE cliente_id = ?` ✅
* `WHERE cliente_id = ? AND creado_en > ?` ✅
* `WHERE cliente_id = ? ORDER BY creado_en DESC` ✅

**No** sirve bien para `WHERE creado_en > ?` solo, porque el índice está ordenado primero por cliente. Funciona como una guía telefónica ordenada por apellido y nombre: no te sirve para buscar a todos los que se llaman "Juan".

### Los índices no son gratis

* Cada `INSERT` y `UPDATE` tiene que actualizar también cada índice.
* Ocupan disco y memoria.

Regla práctica: crea índices para las consultas que **realmente** haces. Las claves foráneas casi siempre necesitan uno, porque Postgres **no** las indexa solo. Para saber si una consulta usa el índice, ejecútala con `EXPLAIN ANALYZE`.

### Índices como reglas de negocio

Un índice único parcial expresa reglas que un `UNIQUE` simple no puede:

```sql
-- Un cliente puede tener muchos pedidos, pero solo uno PENDIENTE a la vez
CREATE UNIQUE INDEX uq_un_pendiente_por_cliente
  ON pedidos (cliente_id)
  WHERE estado = 'PENDIENTE';
```

-----

## Resumen en 5 líneas

1. Las tablas se conectan por claves: la primaria identifica la fila, la foránea apunta a otra tabla.
2. Los constraints (`NOT NULL`, `UNIQUE`, `CHECK`, `FOREIGN KEY`) son la última defensa: el dominio da buenos mensajes, la base garantiza los datos.
3. Normalizar es guardar cada dato en un solo lugar; desnormalizar a propósito vale para datos históricos.
4. Un índice acelera las búsquedas por sus columnas; en los compuestos, el orden de las columnas importa.
5. Los índices cuestan en escritura: créalos para consultas reales y verifícalos con `EXPLAIN ANALYZE`.

-----

## Siguiente lección

[2. Transacciones y aislamiento](02-transacciones-y-aislamiento.md): qué pasa cuando varias operaciones tienen que ocurrir juntas.
