# Glosario

Las palabras que aparecen en las lecciones de esta carpeta, explicadas de forma simple. Están en orden alfabético.

-----

**Cambio incompatible (breaking change).** Un cambio en la API que rompe a los clientes que ya existen: quitar o renombrar un campo, cambiar un tipo.

**Code-first.** Escribir primero el código y generar el spec de OpenAPI a partir de él.

**Código de estado.** Número de tres dígitos que resume el resultado de una petición: `200`, `404`, `500`.

**Colección.** El conjunto de recursos de un tipo, como `/pedidos`.

**Contract-first.** Escribir primero el spec de OpenAPI y después el código.

**Contrato.** Lo que la API promete: rutas, campos, tipos y códigos de estado.

**Cookie.** Dato que el servidor le pide al navegador que guarde y reenvíe automáticamente en cada petición a ese dominio.

**CORS.** Mecanismo por el que un servidor le permite al navegador hacer peticiones desde otro origen.

**CSRF.** Ataque en el que otro sitio hace que el navegador del usuario mande una petición a tu API con sus cookies.

**Cuerpo (body).** El contenido que viaja después de los headers, normalmente JSON.

**Cursor opaco.** Token de paginación que el cliente devuelve tal cual, sin interpretar su contenido.

**`ETag`.** Header con la versión de un recurso; con `If-None-Match` sirve para caché y con `If-Match` para bloqueo optimista.

**Filtro.** Condición que restringe qué elementos devuelve una lista.

**Header.** Par `Nombre: valor` con información sobre la petición o la respuesta.

**Idempotente.** Que repetir la operación deja el mismo estado que hacerla una vez.

**ISO 8601.** Formato estándar para fechas y horas: `2026-10-12T14:30:00Z`.

**Lista blanca (whitelist).** Lista cerrada de valores permitidos; todo lo demás se rechaza.

**Método.** El verbo de la petición: `GET`, `POST`, `PUT`, `PATCH`, `DELETE`.

**OpenAPI.** Especificación estándar para describir APIs HTTP. Antes se llamaba Swagger.

**Origen.** La combinación de esquema, dominio y puerto: `https://app.tienda.com:443`.

**Paginación por cursor (keyset).** "Dame las siguientes M después de esta fila", usando los valores de la última fila vista.

**Paginación por offset.** "Salteá N filas y dame las siguientes M".

**Política del mismo origen.** Regla del navegador que impide que un script lea respuestas de otro origen sin permiso.

**Preflight.** Petición `OPTIONS` que el navegador manda antes de la real para preguntar si CORS la permite.

**Recurso.** Una cosa del negocio expuesta por la API, con su URL: un pedido, un cliente.

**Schema.** La descripción de la forma de un objeto JSON: campos, tipos, obligatorios.

**Seguro (método).** Método que no cambia nada en el servidor, como `GET`.

**Sin estado (stateless).** El servidor no recuerda peticiones anteriores; cada una trae todo lo necesario.

**Spec.** El archivo OpenAPI de una API (`openapi.yaml`).

**Swagger UI.** Interfaz web que muestra el spec de OpenAPI como documentación interactiva.
