# OpenAPI y Swagger

## En una frase

OpenAPI es un formato estándar (YAML o JSON) para describir una API HTTP completa (rutas, parámetros, cuerpos, respuestas y seguridad); con ese archivo se genera documentación navegable (Swagger UI), código de clientes y servidores, y tests que verifican que la API cumple lo que promete.

-----

## Antes de empezar

Conviene que ya sepas:

* Diseño de endpoints y DTOs: [Diseño REST](02-diseno-rest.md).
* Paginación: [Paginación y filtros](03-paginacion-y-filtros.md).

Palabras nuevas (también están en el [Glosario](Glosario.md)):

* **OpenAPI:** Especificación estándar para describir APIs HTTP. Antes se llamaba Swagger.
* **Spec:** El archivo OpenAPI de tu API (`openapi.yaml`).
* **Swagger UI:** Interfaz web que muestra el spec como documentación interactiva.
* **Contract-first:** Escribir primero el spec y después el código.
* **Code-first:** Escribir primero el código y generar el spec a partir de él.
* **Schema:** La descripción de la forma de un objeto JSON: campos, tipos, obligatorios.

-----

## El problema: documentación que miente

La documentación escrita a mano en una wiki se desactualiza el primer día. Alguien agrega un campo, cambia un código de error, y nadie actualiza el documento. Los equipos de frontend y mobile terminan leyendo el código del backend, o preguntando por chat.

OpenAPI convierte el contrato en un **archivo que las máquinas entienden**, y que por lo tanto se puede verificar.

## Cómo se ve un spec

```yaml
openapi: 3.1.0
info:
  title: API de la tienda
  version: 1.0.0

servers:
  - url: https://api.tienda.com/api/v1

paths:
  /pedidos:
    post:
      summary: Crear un pedido
      operationId: crearPedido
      security:
        - bearerAuth: []
      requestBody:
        required: true
        content:
          application/json:
            schema:
              $ref: '#/components/schemas/CrearPedidoRequest'
      responses:
        '201':
          description: Pedido creado
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/PedidoResponse'
        '401':
          $ref: '#/components/responses/NoAutenticado'
        '409':
          description: Sin stock
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/ErrorResponse'

components:
  securitySchemes:
    bearerAuth:
      type: http
      scheme: bearer
      bearerFormat: JWT

  schemas:
    CrearPedidoRequest:
      type: object
      required: [lineas]
      properties:
        lineas:
          type: array
          minItems: 1
          items:
            $ref: '#/components/schemas/LineaRequest'

    PedidoResponse:
      type: object
      required: [id, estado, total, creadoEn]
      properties:
        id: { type: string }
        estado: { type: string, enum: [PENDIENTE, ENVIADO, CANCELADO] }
        total: { type: string, example: "150.00" }
        creadoEn: { type: string, format: date-time }

    ErrorResponse:
      type: object
      required: [codigo, mensaje]
      properties:
        codigo: { type: string }
        mensaje: { type: string }
        callId: { type: string }
```

* **`paths`:** cada ruta con sus métodos, parámetros, cuerpo y respuestas posibles.
* **`components`:** piezas reutilizables (schemas, respuestas, esquemas de seguridad) que se referencian con `$ref`.
* **`security`:** qué autenticación pide cada operación.

Fíjate en que el spec documenta también los **errores**, no solo el caso feliz. Para el cliente, saber que puede recibir un `409` con `codigo: SIN_STOCK` es tan importante como el `201`.

## Swagger UI

Con el spec, Swagger UI muestra una página navegable donde puedes ver cada endpoint y **probarlo** desde el navegador. En Ktor se sirve con una línea, como viste en [Ktor server y plugins](../14-ktor-y-koin/01-ktor-server-y-plugins.md):

```kotlin
routing {
  swaggerUI(path = "swagger", swaggerFile = "openapi/documentation.yaml")
}
```

Decide si exponerlo en producción: es cómodo para integradores, pero también le muestra tu API completa a cualquiera. Una opción común es exponerlo solo en entornos internos.

## Contract-first vs code-first

| | Contract-first | Code-first |
| --- | --- | --- |
| Orden | Spec → código | Código → spec generado |
| El contrato se discute | Antes de programar, con frontend y mobile | Después, cuando ya está hecho |
| Riesgo | El código se aparta del spec | El spec refleja detalles de implementación |
| Herramientas | Generadores de código a partir del spec | Librerías que leen las rutas y los DTOs |

**Contract-first** encaja muy bien cuando varios equipos dependen de la API: se acuerda el spec, y el frontend puede empezar a trabajar contra un servidor simulado mientras el backend implementa.

La plantilla [ktor-hexagonal-multimodule](https://github.com/mobiletoly/ktor-hexagonal-multimodule) usa este enfoque: los modelos REST (los DTOs) se **generan** a partir del spec de OpenAPI, así el código no puede apartarse del contrato en la forma de los datos.

**Code-first** es más rápido para empezar. Hay librerías para Ktor que generan el spec a partir de las rutas, y las versiones recientes de Ktor traen soporte propio; revisa la documentación de tu versión.

## Generar código

Con **OpenAPI Generator** (u otras herramientas) a partir del spec se generan:

* **Modelos** (DTOs con `@Serializable`) para el servidor.
* **Clientes** en Kotlin, TypeScript, Swift... para el frontend y mobile: llaman a tu API con funciones tipadas, sin escribir HTTP a mano.
* **Servidores simulados (mocks)** que responden los ejemplos del spec.

Lo generado **no se edita a mano**: se regenera en el build cada vez que cambia el spec.

## Mantener el spec sincronizado

El peor spec es uno que miente. Defensas, de menor a mayor esfuerzo:

1. **Revisarlo en el code review:** todo cambio de API incluye el cambio del spec en el mismo merge request.
2. **Generar los DTOs desde el spec** (contract-first): el código no compila si no coincide.
3. **Tests de contrato:** en los tests con [`testApplication`](../19-testing/04-test-application.md), validar que cada respuesta cumple el schema del spec.
4. **Detectar cambios incompatibles en el CI:** hay herramientas que comparan el spec nuevo con el anterior y fallan si quitaste un campo o cambiaste un tipo, que son los cambios de [Diseño REST](02-diseno-rest.md) que rompen clientes.

-----

## Resumen en 5 líneas

1. OpenAPI describe la API en un archivo: rutas, parámetros, cuerpos, respuestas (incluidos los errores) y seguridad.
2. `components` y `$ref` reutilizan schemas; Swagger UI lo muestra como documentación interactiva.
3. Contract-first acuerda el spec antes de programar y genera los DTOs; code-first genera el spec desde el código.
4. Desde el spec se generan clientes tipados para frontend y mobile, y servidores simulados.
5. Un spec que miente es peor que ninguno: revísalo en cada cambio, valida respuestas en tests y detecta cambios incompatibles en el CI.

-----

## Siguiente lección

Terminaste HTTP y APIs. Si todavía no la leíste, sigue con [Ktor y Koin](../14-ktor-y-koin/README.md). Vuelve al índice: [README principal](../README.md).
