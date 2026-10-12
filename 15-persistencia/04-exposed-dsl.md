# Exposed DSL

## En una frase

Exposed es una librería de JetBrains para escribir SQL en Kotlin con tipos: las tablas son objetos, las columnas son propiedades, y si te equivocas de tipo o de nombre, no compila.

-----

## Antes de empezar

Conviene que ya sepas:

* Los adapters secundarios y los modelos separados: [Arquitectura hexagonal](../12-arquitectura/02-arquitectura-hexagonal.md).
* Transacciones: [Transacciones y aislamiento](02-transacciones-y-aislamiento.md).
* El pool de conexiones: [PostgreSQL y HikariCP](03-postgresql-y-hikaricp.md).

Palabras nuevas (también están en el [Glosario](Glosario.md)):

* **Exposed:** Librería de acceso a bases de datos para Kotlin, con dos estilos: DSL y DAO.
* **DSL de Exposed:** Estilo que se parece a SQL: `selectAll().where { ... }`.
* **DAO de Exposed:** Estilo con clases entidad que se cargan y se guardan como objetos.
* **`ResultRow`:** Una fila del resultado de una consulta.
* **Problema N+1:** Hacer una consulta para traer N elementos y después una consulta más por cada uno.

-----

## Conectar Exposed al pool

```kotlin
val database = Database.connect(crearDataSource(config))
```

Exposed no abre conexiones por su cuenta: las pide al `DataSource` de HikariCP.

> **Versiones:** desde Exposed 1.0, los paquetes cambiaron a `org.jetbrains.exposed.v1.*`. Los ejemplos de esta lección no muestran imports; revisa la guía de migración de la versión que uses.

## Definir una tabla

```kotlin
object PedidosTable : Table("pedidos") {
  val id = uuid("id")
  val clienteId = uuid("cliente_id")
  val total = decimal("total", precision = 12, scale = 2)
  val estado = varchar("estado", 20)
  val version = integer("version").default(0)
  val creadoEn = timestamp("creado_en")

  override val primaryKey = PrimaryKey(id)
}
```

El objeto `PedidosTable` **describe** una tabla que ya existe; no la crea. Quien crea y modifica las tablas en producción es Flyway ([lección 5](05-migraciones-con-flyway.md)).

## Las operaciones básicas

Todas las operaciones van dentro de una transacción:

```kotlin
transaction(database) {
  // INSERT
  PedidosTable.insert {
    it[id] = UUID.randomUUID()
    it[clienteId] = cliente
    it[total] = BigDecimal("150.00")
    it[estado] = "PENDIENTE"
    it[creadoEn] = Clock.System.now()
  }

  // SELECT ... WHERE
  val pendientes = PedidosTable
    .selectAll()
    .where { (PedidosTable.clienteId eq cliente) and (PedidosTable.estado eq "PENDIENTE") }
    .orderBy(PedidosTable.creadoEn, SortOrder.DESC)
    .limit(20)
    .toList()

  // UPDATE: devuelve cuántas filas cambió
  val filas = PedidosTable.update({ PedidosTable.id eq pedidoId }) {
    it[estado] = "CANCELADO"
  }

  // DELETE
  PedidosTable.deleteWhere { PedidosTable.id eq pedidoId }
}
```

Fíjate en que `update` devuelve un `Int`: la cantidad de filas afectadas. Ese número es la base del bloqueo optimista y del compare-and-swap ([lección 6](06-bloqueos-y-concurrencia.md)).

## El repositorio completo

En una arquitectura hexagonal, Exposed vive **solo** dentro del adapter. El mapeo de `ResultRow` a dominio también, como extensión privada:

```kotlin
class ExposedPedidoRepository(
  private val database: Database,
) : PedidoRepository {

  override suspend fun buscar(id: PedidoId): Pedido? = dbQuery {
    PedidosTable
      .selectAll()
      .where { PedidosTable.id eq id.value }
      .singleOrNull()
      ?.toPedido()
  }

  override suspend fun guardar(pedido: Pedido) = dbQuery {
    PedidosTable.insert {
      it[id] = pedido.id.value
      it[clienteId] = pedido.clienteId.value
      it[total] = pedido.total
      it[estado] = pedido.estado.name
      it[creadoEn] = pedido.creadoEn
    }
    Unit
  }

  private suspend fun <T> dbQuery(bloque: Transaction.() -> T): T =
    newSuspendedTransaction(Dispatchers.IO, database) { bloque() }
}

private fun ResultRow.toPedido() = Pedido(
  id = PedidoId(this[PedidosTable.id]),
  clienteId = ClienteId(this[PedidosTable.clienteId]),
  total = this[PedidosTable.total],
  estado = EstadoPedido.valueOf(this[PedidosTable.estado]),
  creadoEn = this[PedidosTable.creadoEn],
)
```

Puntos clave:

* `PedidosTable` y `ResultRow` **nunca** salen del adapter. El puerto devuelve `Pedido`.
* `toPedido()` es privada: nadie fuera del repositorio necesita saber cómo se ve una fila.
* `dbQuery` centraliza "transacción + `Dispatchers.IO`". En Exposed 1.0, el equivalente de `newSuspendedTransaction` es `suspendTransaction`.

## DSL o DAO

Exposed también tiene un estilo DAO:

```kotlin
class PedidoEntity(id: EntityID<UUID>) : UUIDEntity(id) {
  companion object : UUIDEntityClass<PedidoEntity>(PedidosDaoTable)
  var estado by PedidosDaoTable.estado
}
```

Es cómodo, pero la entidad DAO es un objeto **vivo** atado a la transacción: si la devuelves fuera del adapter, el dominio termina dependiendo de Exposed. Con el DSL, el mapeo explícito a una data class de dominio es obligatorio, y eso protege la arquitectura. Por eso, en un proyecto hexagonal, se suele elegir el **DSL**.

## Ver el SQL que se ejecuta

```kotlin
transaction {
  addLogger(StdOutSqlLogger)
  // ...
}
```

Úsalo en desarrollo para entender qué hace Exposed y detectar consultas de más.

## El problema N+1

```kotlin
val pedidos = PedidosTable.selectAll().map { it.toPedido() }      // 1 consulta
pedidos.forEach { pedido ->
  val lineas = LineasTable.selectAll()
    .where { LineasTable.pedidoId eq pedido.id.value }.toList()  // N consultas
}
```

Con 100 pedidos son 101 consultas. La solución es traer todo en una o dos:

```kotlin
val ids = pedidos.map { it.id.value }
val lineasPorPedido = LineasTable
  .selectAll()
  .where { LineasTable.pedidoId inList ids }
  .groupBy { it[LineasTable.pedidoId] }                          // 1 consulta más
```

O con un `join` si lo necesitas en una sola consulta.

## Exposed y los tests

`SchemaUtils.create(PedidosTable)` crea la tabla a partir del objeto. Es útil en un prototipo, pero en producción y en los tests de integración conviene correr **las mismas migraciones de Flyway** contra un Postgres real (Testcontainers, en `19-testing`). Si no, el test prueba un esquema distinto al real.

-----

## Resumen en 5 líneas

1. Exposed escribe SQL tipado en Kotlin; las tablas son objetos `*Table` que describen el esquema.
2. Toda operación va dentro de `transaction { }`; `update` devuelve las filas afectadas, clave para el control de concurrencia.
3. Exposed, `*Table` y `ResultRow` viven solo en el adapter; el mapeo a dominio es una extensión privada `ResultRow.toX()`.
4. En hexagonal se prefiere el DSL sobre el DAO, porque obliga a mapear a modelos de dominio limpios.
5. Desde código `suspend`, usa una transacción suspendida en `Dispatchers.IO`, y vigila el problema N+1.

-----

## Siguiente lección

[5. Migraciones con Flyway](05-migraciones-con-flyway.md): cómo cambia el esquema a lo largo del tiempo.
