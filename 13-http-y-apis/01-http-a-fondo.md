# HTTP a fondo

## En una frase

HTTP es un protocolo de texto de **petición y respuesta**: el cliente manda un método, una ruta, headers y a veces un cuerpo; el servidor responde con un código de estado, headers y a veces un cuerpo, y no recuerda nada entre una petición y la siguiente.

-----

## Antes de empezar

No necesitas conocimientos previos de esta wiki.

Palabras nuevas (también están en el [Glosario](Glosario.md)):

* **Método:** El verbo de la petición: `GET`, `POST`, `PUT`, `PATCH`, `DELETE`.
* **Código de estado:** Número de tres dígitos que resume el resultado: `200`, `404`, `500`.
* **Header:** Par `Nombre: valor` con información sobre la petición o la respuesta.
* **Cuerpo (body):** El contenido que viaja después de los headers, normalmente JSON.
* **Sin estado (stateless):** El servidor no recuerda peticiones anteriores; cada una trae todo lo necesario.
* **Origen:** La combinación de esquema, dominio y puerto: `https://app.tienda.com:443`.
* **CORS:** Mecanismo por el que un servidor le permite al navegador hacer peticiones desde otro origen.

-----

## Una petición y una respuesta, de verdad

Lo que viaja por la red es **texto** (en HTTP/1.1; HTTP/2 y 3 lo codifican en binario, pero el significado es el mismo):

```
POST /api/v1/pedidos HTTP/1.1
Host: api.tienda.com
Content-Type: application/json
Accept: application/json
Authorization: Bearer eyJhbGciOiJSUzI1NiJ9...
X-Request-Id: 8f3c2a10

{"lineas":[{"productoId":"prod-1","cantidad":2}]}
```

```
HTTP/1.1 201 Created
Content-Type: application/json
Location: /api/v1/pedidos/p-9
X-Request-Id: 8f3c2a10

{"id":"p-9","estado":"PENDIENTE","total":"150.00"}
```

* **Primera línea:** método, ruta y versión; o versión, código y frase.
* **Headers:** metadatos.
* **Línea vacía**, y después el **cuerpo**.

**HTTP no tiene memoria.** El servidor no sabe que la petición anterior fue del mismo usuario. Por eso cada petición trae su token en `Authorization`: lo viste en [JWT con RS256](../16-seguridad/02-jwt-rs256.md).

## Métodos

| Método | Para qué | ¿Seguro? | ¿Idempotente? | ¿Cuerpo? |
| --- | --- | --- | --- | --- |
| `GET` | Leer | ✅ | ✅ | No |
| `POST` | Crear o ejecutar una acción | ❌ | ❌ | Sí |
| `PUT` | Reemplazar completo | ❌ | ✅ | Sí |
| `PATCH` | Modificar en parte | ❌ | Depende | Sí |
| `DELETE` | Borrar | ❌ | ✅ | No suele |
| `OPTIONS` | Preguntar qué se permite (lo usa CORS) | ✅ | ✅ | No |

* **Seguro:** no cambia nada en el servidor. Un `GET` que borra datos es un bug grave: navegadores, buscadores y proxies hacen `GET` por su cuenta (para precargar o cachear).
* **Idempotente:** repetirlo deja el mismo estado. Lo viste en [Idempotencia](../17-resiliencia-e-integraciones/02-idempotencia.md).

## Códigos de estado

La primera cifra dice de quién es la culpa:

| Familia | Significa |
| --- | --- |
| `2xx` | Éxito |
| `3xx` | Redirección o "usa tu copia" |
| `4xx` | Error del **cliente**: reintentar igual no sirve |
| `5xx` | Error del **servidor**: puede ser pasajero |

Los que vas a usar:

| Código | Cuándo |
| --- | --- |
| `200 OK` | Éxito con cuerpo |
| `201 Created` | Se creó un recurso; con header `Location` apuntando a él |
| `204 No Content` | Éxito sin cuerpo (por ejemplo, un `DELETE`) |
| `304 Not Modified` | El cliente ya tiene la versión actual (caché) |
| `400 Bad Request` | La petición está mal formada: JSON inválido, campo con formato incorrecto |
| `401 Unauthorized` | No autenticado: falta el token o no es válido. (Mal nombre: debería ser "unauthenticated") |
| `403 Forbidden` | Autenticado, pero sin permiso |
| `404 Not Found` | No existe, o no quieres revelar que existe |
| `405 Method Not Allowed` | La ruta existe, pero no con ese método |
| `409 Conflict` | Choca con el estado actual: ya enviado, versión vieja, duplicado |
| `415 Unsupported Media Type` | El `Content-Type` no es el esperado |
| `422 Unprocessable Content` | La forma es correcta, pero el contenido no tiene sentido |
| `429 Too Many Requests` | Superó el límite; con `Retry-After` |
| `500 Internal Server Error` | Error inesperado del servidor |
| `502` / `503` / `504` | Problemas de un servidor intermedio, servicio no disponible o timeout |

`400` vs `422`: hay equipos que usan solo `400` para todo error de validación. Lo importante es ser **consistente** en toda la API.

## Headers importantes

| Header | Dirección | Para qué |
| --- | --- | --- |
| `Content-Type` | Ambas | Formato del cuerpo: `application/json` |
| `Accept` | Petición | Qué formato acepta el cliente en la respuesta |
| `Authorization` | Petición | Credenciales: `Bearer <token>` |
| `Location` | Respuesta | Dónde está el recurso creado (con `201`) |
| `Retry-After` | Respuesta | Cuándo reintentar (con `429` o `503`) |
| `Cache-Control` | Respuesta | Si se puede cachear y por cuánto: `no-store`, `max-age=60` |
| `ETag` | Respuesta | Versión del recurso, como una huella |
| `If-None-Match` | Petición | "Dame el recurso solo si cambió desde este `ETag`" → `304` si no cambió |
| `If-Match` | Petición | "Modifícalo solo si sigue siendo esta versión" → `412` si cambió |
| `X-Request-Id` | Ambas | Identificador para seguir la petición en los logs |

`If-Match` con `ETag` es el **bloqueo optimista** de [Bloqueos y concurrencia](../15-persistencia/06-bloqueos-y-concurrencia.md), pero llevado a HTTP: el cliente dice qué versión editó, y si alguien la cambió mientras tanto, el servidor responde `412 Precondition Failed`.

Una regla de seguridad: las respuestas con datos privados llevan `Cache-Control: no-store`, para que ningún proxy ni navegador las guarde.

## Cookies

Una cookie es un dato que el **servidor** le pide al navegador que guarde y le reenvíe **automáticamente** en cada petición a ese dominio:

```
HTTP/1.1 200 OK
Set-Cookie: refresh_token=a8f3...; HttpOnly; Secure; SameSite=Strict; Path=/auth; Max-Age=2592000
```

| Atributo | Efecto |
| --- | --- |
| `HttpOnly` | JavaScript no puede leerla: un script malicioso inyectado en la página no la roba |
| `Secure` | Solo viaja por HTTPS |
| `SameSite=Strict` | No se envía en peticiones que vienen de otros sitios |
| `SameSite=Lax` | Solo se envía desde otros sitios en navegación simple (clic en un link) |
| `Path=/auth` | Solo se envía a rutas que empiezan con `/auth` |
| `Max-Age` | Segundos hasta que vence |

Que el navegador la mande **solo** es una ventaja y un peligro: si otro sitio hace que el navegador del usuario mande una petición a tu API, la cookie viaja igual. Eso es **CSRF**. `SameSite` es la defensa principal.

Por eso, para una web, el refresh token va en una cookie `HttpOnly; Secure; SameSite=Strict` limitada a `/auth`, como viste en [Refresh tokens](../16-seguridad/04-refresh-tokens.md). Las apps móviles no usan cookies: guardan los tokens en almacenamiento seguro y los mandan en headers.

## CORS

### El problema que resuelve

El navegador aplica la **política del mismo origen**: un script cargado desde `https://app.tienda.com` no puede leer respuestas de `https://api.tienda.com`, porque son orígenes distintos (cambia el dominio). Sin esa regla, cualquier sitio que visites podría leer tus datos de otros sitios donde tienes sesión.

CORS es la forma en que el servidor dice: "a **este** origen sí lo dejo".

### Cómo funciona

Para peticiones "no simples" (con `Authorization`, con `Content-Type: application/json`, o con métodos como `PUT` o `DELETE`), el navegador manda **antes** una petición de permiso, el *preflight*:

```
OPTIONS /api/v1/pedidos HTTP/1.1
Origin: https://app.tienda.com
Access-Control-Request-Method: POST
Access-Control-Request-Headers: authorization, content-type
```

```
HTTP/1.1 204 No Content
Access-Control-Allow-Origin: https://app.tienda.com
Access-Control-Allow-Methods: GET, POST, PUT, DELETE
Access-Control-Allow-Headers: authorization, content-type
Access-Control-Max-Age: 600
```

Si la respuesta lo permite, el navegador manda la petición real. Si no, la bloquea y en la consola aparece el famoso error de CORS.

### Lo que hay que entender bien

* **CORS lo aplica el navegador, no el servidor.** `curl`, Postman, una app móvil u otro backend ignoran CORS por completo. **No es una medida de seguridad de tu API**: la seguridad es la autenticación y la autorización.
* **CORS protege al usuario** del navegador, para que otro sitio no lea sus datos.
* Con cookies o credenciales, `Access-Control-Allow-Origin` **no puede** ser `*`: tiene que ser el origen exacto.
* Un error de CORS en desarrollo casi siempre se arregla configurando el servidor (el plugin CORS de Ktor, en [Ktor server y plugins](../14-ktor-y-koin/01-ktor-server-y-plugins.md)), no el frontend.

-----

## Resumen en 5 líneas

1. HTTP es petición y respuesta en texto, sin estado: cada petición trae todo, incluido el token.
2. `GET` es seguro; `GET`, `PUT` y `DELETE` son idempotentes; `POST` no.
3. `2xx` éxito, `4xx` culpa del cliente (`401` no autenticado, `403` sin permiso, `409` conflicto), `5xx` culpa del servidor.
4. Las cookies viajan solas: `HttpOnly`, `Secure` y `SameSite` las protegen del robo por scripts y del CSRF.
5. CORS lo aplica el navegador para proteger al usuario; no protege tu API, y con credenciales requiere el origen exacto.

-----

## Siguiente lección

[2. Diseño REST](02-diseno-rest.md): cómo organizar los endpoints.
