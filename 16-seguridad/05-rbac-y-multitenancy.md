# RBAC y multi-tenancy

## En una frase

Autenticar es saber **quién** eres; autorizar es decidir **qué puedes hacer y dónde**. Con RBAC, los permisos se agrupan en roles, y en un sistema multi-tenant cada rol se asigna **dentro de una empresa**, cuyos datos nunca se mezclan con los de otra.

-----

## Antes de empezar

Conviene que ya sepas:

* Cómo se identifica al usuario: [JWT con RS256](02-jwt-rs256.md).
* Use cases y outcomes: [La capa application](../12-arquitectura/05-capa-application.md).

Palabras nuevas (también están en el [Glosario](Glosario.md)):

* **Autenticación:** Verificar quién eres.
* **Autorización:** Decidir si puedes hacer algo.
* **RBAC (Role-Based Access Control):** Control de acceso basado en roles.
* **Permiso:** Una acción concreta, como `pedidos:cancelar`.
* **Rol:** Un conjunto de permisos con nombre, como `ADMIN` u `OPERADOR`.
* **Tenant:** Un cliente del sistema (una empresa) cuyos datos están aislados de los demás.
* **Multi-tenancy:** Una sola aplicación que sirve a muchos tenants.
* **Entitlement:** Una funcionalidad que una empresa tiene contratada.

-----

## Autenticación ≠ autorización

| | Pregunta | Error HTTP si falla |
| --- | --- | --- |
| Autenticación | ¿Quién eres? | `401 Unauthorized` |
| Autorización | ¿Puedes hacer esto? | `403 Forbidden` |

El JWT resuelve la primera. La segunda es lógica de negocio.

## RBAC: roles y permisos

```
Usuario ──> Rol ──> Permisos
  Ana       ADMIN     pedidos:ver, pedidos:cancelar, usuarios:invitar
  Juan      OPERADOR  pedidos:ver
```

La regla de oro: **el código pregunta por permisos, no por roles**.

```kotlin
// ❌ Si mañana creas el rol SUPERVISOR, hay que buscar todos los ifs
if (usuario.rol == Rol.ADMIN) { ... }

// ✅ El rol es configuración; el permiso es lo que el código necesita
if (Permiso.CANCELAR_PEDIDOS in contexto.permisos) { ... }
```

```kotlin
enum class Permiso { VER_PEDIDOS, CANCELAR_PEDIDOS, INVITAR_USUARIOS }

enum class Rol(val permisos: Set<Permiso>) {
  ADMIN(Permiso.entries.toSet()),
  OPERADOR(setOf(Permiso.VER_PEDIDOS)),
}
```

## Multi-tenancy: el rol depende de la empresa

En un SaaS, Ana puede ser `ADMIN` en la empresa A y `OPERADOR` en la empresa B. El rol no es una propiedad del usuario: es una **asignación**.

```kotlin
data class RoleAssignment(
  val usuarioId: UsuarioId,
  val empresaId: EmpresaId,
  val rol: Rol,
)
```

```
Ana ──┬── empresa A ──> ADMIN
      └── empresa B ──> OPERADOR
```

## El contexto de autorización

En cada petición se arma un objeto con todo lo necesario para autorizar:

```kotlin
data class AuthContext(
  val usuarioId: UsuarioId,
  val empresaId: EmpresaId,
  val permisos: Set<Permiso>,
)
```

Se construye en el adapter (la ruta) a partir del JWT y de las asignaciones, y se pasa al use case dentro del command. El use case autoriza y devuelve un motivo explícito si niega:

```kotlin
sealed interface DenyReason {
  data object SinPermiso : DenyReason
  data object NoPerteneceALaEmpresa : DenyReason
  data object FuncionalidadNoContratada : DenyReason
}

sealed interface CancelarPedidoOutcome {
  data class Cancelado(val pedido: Pedido) : CancelarPedidoOutcome
  data class Denegado(val motivo: DenyReason) : CancelarPedidoOutcome
  data object NoEncontrado : CancelarPedidoOutcome
}
```

¿Por qué en la capa application y no solo en la ruta? Porque la autorización es una **regla de negocio**: si mañana el mismo use case se dispara desde un job o desde otra API, la regla tiene que seguir valiendo.

## La regla más importante: el tenant nunca viene del cliente

```kotlin
// ❌ El cliente puede mandar la empresa que quiera
val empresaId = call.receive<CrearPedidoRequest>().empresaId
```

Si confías en el `empresaId` del body, un usuario de la empresa A crea pedidos en la empresa B cambiando un número. El tenant se obtiene del **token**, o de la URL **verificada** contra las asignaciones del usuario:

```kotlin
// ✅ La empresa de la URL se verifica contra las asignaciones del usuario
val empresaId = EmpresaId(call.parameters["empresaId"]!!)
val asignacion = asignaciones.buscar(usuarioId, empresaId)
  ?: return call.respond(HttpStatusCode.Forbidden)
```

## Aislamiento de datos: toda consulta filtra por tenant

Cada tabla con datos de un tenant tiene su columna `empresa_id`, y **toda** consulta la filtra:

```kotlin
PedidosTable
  .selectAll()
  .where { (PedidosTable.id eq pedidoId) and (PedidosTable.empresaId eq contexto.empresaId) }
```

Si te olvidas del filtro **una sola vez**, un tenant ve los datos de otro. Defensas:

* El puerto del repositorio **exige** el `empresaId` en cada método, así no se puede olvidar.
* Índices que empiezan por `empresa_id`, que además aceleran estas consultas.
* Tests de integración que crean dos empresas y verifican que una no ve lo de la otra.
* Como capa extra, Postgres permite *Row-Level Security*: políticas que filtran por tenant dentro de la propia base.

## ¿403 o 404?

Si el usuario de la empresa A pide el pedido `p-99`, que es de la empresa B:

* Con `403 Forbidden` confirmas que `p-99` **existe**.
* Con `404 Not Found` no revelas nada.

Para recursos de **otros** tenants, responde `404`: para el usuario, simplemente no existe. Reserva `403` para cuando el recurso es de su empresa pero le falta el permiso.

## Entitlements: lo que la empresa contrató

Además de los permisos del usuario, la empresa puede tener funcionalidades contratadas o no (reportes avanzados, integraciones). Es otra condición del mismo tipo: el use case la revisa y, si falta, devuelve `Denegado(FuncionalidadNoContratada)`.

-----

## Resumen en 5 líneas

1. Autenticar es saber quién eres (`401`); autorizar es decidir qué puedes hacer (`403`).
2. RBAC agrupa permisos en roles, pero el código pregunta por permisos, no por roles.
3. En multi-tenancy el rol es una asignación por empresa, y cada petición arma un `AuthContext`.
4. La autorización es una regla de negocio: vive en el use case y devuelve un motivo explícito al negar.
5. El tenant nunca se toma del body; toda consulta filtra por `empresa_id`, y los recursos de otro tenant responden `404`.

-----

## Siguiente lección

[6. Tokens de servicio y rate limiting](06-tokens-de-servicio-y-rate-limiting.md): cuando quien llama no es una persona.
