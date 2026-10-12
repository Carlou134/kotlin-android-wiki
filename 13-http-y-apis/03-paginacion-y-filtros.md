# Paginación y filtros

## En una frase

Una lista nunca se devuelve entera: se devuelve **por páginas**, con un tamaño máximo, y se filtra y ordena con parámetros que el servidor valida contra una lista cerrada; la forma de paginar (offset o cursor) define cuánto escala y qué tan consistente es.

-----

## Antes de empezar

Conviene que ya sepas:

* Diseño de endpoints: [Diseño REST](02-diseno-rest.md).
* Índices compuestos y el orden de sus columnas: [SQL relacional](../15-persistencia/01-sql-relacional.md).
* Consultas con Exposed: [Exposed DSL](../15-persistencia/04-exposed-dsl.md).

Palabras nuevas (también están en el [Glosario](Glosario.md)):

* **Paginación por offset:** "Salteá N filas y dame las siguientes M".
* **Paginación por cursor (keyset):** "Dame las siguientes M después de esta fila", usando los valores de la última fila vista.
* **Cursor opaco:** Un token que el cliente devuelve tal cual, sin entender su contenido.
* **Filtro:** Condición que restringe qué elementos devuelve una lista.
* **Lista blanca (whitelist):** Lista cerrada de valores permitidos; todo lo demás se rechaza.

-----

## El problema: listas que crecen

`GET /pedidos` funciona con 50 pedidos. Con 2 millones:

* La consulta tarda segundos y ocupa memoria en la base.
* La respuesta pesa cientos de MB.
* El cliente no puede mostrarla.

Toda lista que puede crecer se pagina **desde el primer día**. Agregar paginación después es un cambio incompatible.

## Paginación por offset

```
GET /api/v1/pedidos?pagina=3&tamano=20
```

```sql
SELECT * FROM pedidos
WHERE empresa_id = ?
ORDER BY creado_en DESC, id DESC
LIMIT 20 OFFSET 40;
```

```json
{
  "items": [ ... ],
  "pagina": 3,
  "tamano": 20,
  "total": 1834
}
```

**Ventajas:** simple, permite saltar a cualquier página ("ir a la página 47"), y es fácil de mostrar en una tabla con números de página.

**Problemas:**

* **Offsets profundos son lentos:** `OFFSET 100000` obliga a la base a leer y descartar 100.000 filas.
* **Inconsistente si cambian los datos:** si entre la página 1 y la 2 se crea un pedido nuevo, todo se corre un lugar y un elemento aparece **repetido** en la página 2. Si se borra uno, otro se **saltea**.
* **`total` cuesta:** contar 2 millones de filas en cada petición es caro. Considera omitirlo o calcularlo aparte.

## Paginación por cursor (keyset)

En vez de "saltear N", se dice "dame lo que viene **después de** este elemento":

```
GET /api/v1/pedidos?limite=20
GET /api/v1/pedidos?limite=20&cursor=eyJjIjoiMjAyNi0xMC0xMlQxNDozMDowMFoiLCJpIjoicC05In0
```

El cursor codifica los valores del **último** elemento de la página anterior (en este ejemplo, su `creado_en` y su `id`, en Base64). La consulta:

```sql
SELECT * FROM pedidos
WHERE empresa_id = ?
  AND (creado_en, id) < (?, ?)          -- después del último visto
ORDER BY creado_en DESC, id DESC
LIMIT 21;                               -- uno de más
```

* **El `id` desempata:** si dos pedidos tienen el mismo `creado_en`, sin desempate uno podría perderse entre páginas. El orden tiene que ser **único**.
* **`LIMIT 21` para pedir 20:** si vuelven 21, hay página siguiente; se devuelven 20 y el cursor se arma con el vigésimo.
* **Con un índice** sobre `(empresa_id, creado_en, id)`, cada página es igual de rápida, sea la primera o la diez mil.

```json
{
  "items": [ ... ],
  "siguienteCursor": "eyJjIjoi..."
}
```

`siguienteCursor` es `null` en la última página.

**El cursor es opaco:** el cliente no debe interpretarlo ni armarlo, solo devolverlo. Así puedes cambiar su contenido sin romper a nadie.

## ¿Cuál elegir?

| | Offset | Cursor |
| --- | --- | --- |
| Saltar a la página N | ✅ | ❌ |
| Rendimiento en páginas profundas | ❌ Empeora | ✅ Constante |
| Consistencia si cambian los datos | ❌ Repetidos o salteados | ✅ |
| Ideal para | Tablas de administración con pocos miles de filas | Scroll infinito, sincronizaciones, listas grandes |

Para un job que sincroniza o exporta todo, **siempre** cursor.

## Tamaño máximo

```kotlin
val limite = call.request.queryParameters["limite"]?.toIntOrNull()
  ?.coerceIn(1, 100)        // nunca más de 100, aunque pidan 1.000.000
  ?: 20
```

Si no pones un máximo, alguien pide `?limite=1000000` y vuelves al problema inicial.

## Filtros tipados

Los filtros llegan como query parameters:

```
GET /api/v1/pedidos?estado=PENDIENTE&desde=2026-10-01T00:00:00Z&clienteId=c-7
```

En la ruta, se convierten en un objeto **tipado** y se validan, antes de llegar al use case:

```kotlin
data class PedidoFiltro(
  val estado: EstadoPedido? = null,
  val desde: Instant? = null,
  val hasta: Instant? = null,
  val clienteId: ClienteId? = null,
)
```

Un `estado=CUALQUIERA` que no existe en el enum responde `400`, no se ignora en silencio.

En el repositorio, la consulta se arma agregando solo las condiciones presentes:

```kotlin
val consulta = PedidosTable.selectAll().where { PedidosTable.empresaId eq empresaId.value }

filtro.estado?.let { estado -> consulta.andWhere { PedidosTable.estado eq estado.name } }
filtro.desde?.let { desde -> consulta.andWhere { PedidosTable.creadoEn greaterEq desde } }
filtro.clienteId?.let { c -> consulta.andWhere { PedidosTable.clienteId eq c.value } }
```

Exposed genera parámetros SQL, así que los **valores** de los filtros no pueden inyectar SQL.

## Ordenamiento con lista blanca

```
GET /api/v1/pedidos?orden=-creadoEn
```

El campo de orden **nunca** se pasa directo a la consulta. Se traduce desde una lista cerrada:

```kotlin
enum class OrdenPedido(val columna: Column<*>) {
  CREADO_EN(PedidosTable.creadoEn),
  TOTAL(PedidosTable.total),
}
```

Dos motivos:

* **Seguridad:** un nombre de columna armado con texto del usuario es una puerta a la inyección SQL.
* **Rendimiento:** solo permites ordenar por columnas **con índice**. Ordenar 2 millones de filas por una columna sin índice es lento.

## Filtros, orden e índices van juntos

Cada combinación de filtro y orden que ofreces es una consulta que la base tiene que resolver rápido. Antes de agregar un filtro nuevo, pregúntate: ¿hay un índice que lo cubra? Lo viste en [SQL relacional](../15-persistencia/01-sql-relacional.md): un índice sobre `(empresa_id, estado, creado_en)` sirve para "pedidos de la empresa, con este estado, ordenados por fecha".

-----

## Resumen en 5 líneas

1. Toda lista que puede crecer se pagina desde el primer día, con un tamaño máximo forzado por el servidor.
2. Offset es simple y permite saltar páginas, pero es lento en páginas profundas y repite o saltea elementos si cambian los datos.
3. Cursor (keyset) usa los valores de la última fila, con un orden único y `LIMIT n+1`; es rápido y consistente.
4. Los filtros se convierten en un objeto tipado y validado; Exposed los agrega con `andWhere` como parámetros.
5. El orden se elige de una lista blanca de columnas con índice: nunca texto del usuario directo al SQL.

-----

## Siguiente lección

[4. OpenAPI y Swagger](04-openapi-y-swagger.md): documentar el contrato.
