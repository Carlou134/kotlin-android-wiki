# Diseño REST

## En una frase

Una API REST organiza todo en **recursos** (sustantivos en la URL) sobre los que se aplican los métodos HTTP con su significado; el objetivo no es la pureza teórica, sino una API **predecible**, consistente y que pueda cambiar sin romper a sus clientes.

-----

## Antes de empezar

Conviene que ya sepas:

* Métodos y códigos de estado: [HTTP a fondo](01-http-a-fondo.md).
* Por qué los DTOs no son el dominio: [Arquitectura hexagonal](../12-arquitectura/02-arquitectura-hexagonal.md).

Palabras nuevas (también están en el [Glosario](Glosario.md)):

* **Recurso:** Una cosa del negocio expuesta por la API, con su URL: un pedido, un cliente.
* **Colección:** El conjunto de recursos de un tipo: `/pedidos`.
* **Cambio incompatible (breaking change):** Un cambio en la API que rompe a los clientes que ya existen.
* **Contrato:** Lo que la API promete: rutas, campos, tipos, códigos.
* **ISO 8601:** Formato estándar para fechas y horas: `2026-10-12T14:30:00Z`.

-----

## Recursos con sustantivos

```
GET    /api/v1/pedidos            lista de pedidos
POST   /api/v1/pedidos            crear un pedido
GET    /api/v1/pedidos/{id}       un pedido
PUT    /api/v1/pedidos/{id}       reemplazarlo
PATCH  /api/v1/pedidos/{id}       modificarlo en parte
DELETE /api/v1/pedidos/{id}       borrarlo
```

Reglas:

* **Sustantivos en plural**, no verbos: `/pedidos`, no `/obtenerPedidos`. El verbo es el método HTTP.
* **Minúsculas y guiones** para nombres compuestos: `/solicitudes-de-servicio`.
* **Jerarquía** cuando un recurso pertenece a otro: `/pedidos/{id}/lineas`.
* No más de dos o tres niveles. `/empresas/{e}/clientes/{c}/pedidos/{p}/lineas/{l}` es difícil de usar; mejor `/pedidos/{p}/lineas/{l}`.

### El tenant en la ruta

En un sistema multi-tenant, una opción clara es poner la empresa en la ruta:

```
GET /api/v1/empresas/{empresaId}/pedidos
```

Así cada URL dice de qué empresa habla, y el servidor **verifica** que el usuario pertenezca a esa empresa, como viste en [RBAC y multi-tenancy](../16-seguridad/05-rbac-y-multitenancy.md). La otra opción es tomarla solo del token. Lo importante: **nunca** del cuerpo de la petición.

## Acciones que no son CRUD

"Cancelar un pedido" no es crear, leer, actualizar ni borrar: es una **acción de negocio** con reglas. Hay dos estilos:

```
PATCH /pedidos/{id}   { "estado": "CANCELADO" }      // estilo "recurso puro"
POST  /pedidos/{id}/cancelar  { "motivo": "..." }    // estilo "acción"
```

El primero parece más REST, pero esconde la intención: ¿cualquier cambio de estado está permitido?, ¿dónde va el motivo?, ¿qué pasa si mando `ENVIADO`?

El segundo encaja con una arquitectura de use cases: **una acción = un endpoint = un use case** (`CancelarPedido`), con su command y sus outcomes. Es la opción pragmática más común en APIs de negocio.

## `PUT` vs `PATCH`

* **`PUT`** reemplaza el recurso **completo**. Si omites un campo, se borra (o se pone el valor por defecto).
* **`PATCH`** modifica **solo** los campos que mandas.

Con `PATCH` aparece una pregunta difícil: ¿cómo distingues "no mandé el campo" de "quiero ponerlo en `null`"?

```json
{ "telefono": null }   // ¿borrar el teléfono?
{ }                    // ¿no tocar el teléfono?
```

Hay que decidirlo y documentarlo. El estándar *JSON Merge Patch* (RFC 7396) dice: campo ausente = no tocar; campo en `null` = borrar.

## Respuestas por operación

| Operación | Éxito | Errores típicos |
| --- | --- | --- |
| `GET /pedidos/{id}` | `200` con el recurso | `404` |
| `GET /pedidos` | `200` con la lista (vacía si no hay nada, **nunca** `404`) | `400` por filtros inválidos |
| `POST /pedidos` | `201` + `Location` + el recurso creado | `400`, `409`, `422` |
| `POST /pedidos/{id}/cancelar` | `200` con el recurso actualizado | `404`, `403`, `409` |
| `PUT` / `PATCH` | `200` con el recurso actualizado | `400`, `404`, `409`, `412` |
| `DELETE` | `204` | `404` (o `204` también, para que sea idempotente) |

Todos los errores, con el **mismo formato** de [Manejo de errores con StatusPages](../14-ktor-y-koin/02-manejo-de-errores-con-statuspages.md).

## Convenciones del JSON

Decídelas una vez y aplícalas en toda la API:

| Tema | Convención recomendada | Por qué |
| --- | --- | --- |
| Nombres de campos | `camelCase` | Natural en Kotlin y JavaScript |
| Fechas y horas | ISO 8601 en UTC: `"2026-10-12T14:30:00Z"` | Sin ambigüedad de zona horaria |
| Fechas sin hora | `"2026-10-12"` | Un cumpleaños no tiene zona horaria |
| Dinero | String decimal `"150.00"` o entero en centavos `15000` | Un `double` pierde precisión |
| Enums | Strings: `"PENDIENTE"` | Los números no dicen nada |
| IDs | Strings, aunque sean numéricos | JavaScript pierde precisión con enteros grandes |
| Listas vacías | `[]`, nunca `null` | El cliente no tiene que chequear `null` |

## Versionado y cambios incompatibles

Una vez que una app móvil está en los teléfonos de los usuarios, **no puedes obligarla a actualizarse**. Por eso la API tiene que seguir funcionando para clientes viejos.

| Cambio | ¿Rompe? |
| --- | --- |
| Agregar un endpoint | No |
| Agregar un campo opcional a la respuesta | No (si el cliente ignora campos desconocidos) |
| Agregar un campo opcional a la petición | No |
| Agregar un valor nuevo a un enum de la respuesta | **Puede**: el cliente con un `when` cerrado no lo conoce |
| Quitar o renombrar un campo | **Sí** |
| Cambiar el tipo de un campo | **Sí** |
| Hacer obligatorio un campo que era opcional | **Sí** |
| Cambiar el significado de un código de estado | **Sí** |

Los cambios compatibles se hacen en la misma versión. Los incompatibles van en una versión nueva (`/api/v2/...`), y la vieja convive hasta que los clientes migran.

La otra cara: los **clientes** deben ser tolerantes e ignorar campos desconocidos. Es lo que hiciste con `ignoreUnknownKeys = true` en [Ktor Client y APIs de terceros](../17-resiliencia-e-integraciones/03-ktor-client-y-apis-de-terceros.md).

## DTOs propios

```kotlin
@Serializable
data class CrearPedidoRequest(val lineas: List<LineaRequest>)

@Serializable
data class PedidoResponse(val id: String, val estado: String, val total: String, val creadoEn: String)
```

* Lo que entra termina en `Request`, lo que sale en `Response`.
* **Nunca** expongas la entidad de dominio ni la fila de la base: el día que agregues un campo interno, aparecería en la API sin que nadie lo decida. El DTO **es** el contrato; el dominio puede cambiar por dentro sin romperlo.

-----

## Resumen en 5 líneas

1. Recursos con sustantivos en plural, jerarquías cortas, y el método HTTP como verbo.
2. Las acciones de negocio son `POST /recurso/{id}/accion`: una acción, un endpoint, un use case.
3. `PUT` reemplaza todo, `PATCH` modifica en parte; decide qué significa `null` vs ausente.
4. Convenciones fijas de JSON: camelCase, ISO 8601 en UTC, dinero sin `double`, enums como strings, listas vacías.
5. Agregar es compatible; quitar, renombrar o cambiar tipos rompe y va en `/api/v2`; los DTOs son el contrato, nunca el dominio.

-----

## Siguiente lección

[3. Paginación y filtros](03-paginacion-y-filtros.md): cómo devolver listas grandes.
