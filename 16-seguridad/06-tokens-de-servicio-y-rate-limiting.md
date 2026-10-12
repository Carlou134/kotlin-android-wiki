# Tokens de servicio y rate limiting

## En una frase

Cuando quien llama a tu API es otro sistema y no una persona, se autentica con una credencial propia que intercambia por un token corto; y para que nadie (persona o máquina) abuse de la API, se limita cuántas peticiones puede hacer por unidad de tiempo.

-----

## Antes de empezar

Conviene que ya sepas:

* Emitir y verificar JWT: [JWT con RS256](02-jwt-rs256.md).
* Guardar secretos hasheados: [Refresh tokens](04-refresh-tokens.md).

Palabras nuevas (también están en el [Glosario](Glosario.md)):

* **M2M (machine-to-machine):** Comunicación entre sistemas, sin un usuario humano.
* **Client credentials:** Flujo de OAuth 2.0 en el que un servicio se autentica con su propio ID y secreto.
* **App key:** Credencial de larga vida de una aplicación o integración.
* **Scope:** Permiso acotado que lleva un token de servicio, como `pedidos:leer`.
* **Rate limiting:** Limitar cuántas peticiones acepta la API por cliente y por tiempo.
* **Token bucket:** Algoritmo de rate limiting: un balde de fichas que se rellena a ritmo constante.

-----

## Tokens de servicio

Ejemplos de llamadas M2M:

* Un sistema de un cliente que consulta sus pedidos de forma automática.
* Un servicio interno que le pide datos a otro.
* Un proveedor que te notifica eventos por webhook.

No hay usuario ni contraseña ni login. Pero tampoco quieres mandar una credencial de larga vida en **cada** petición: cuantas más veces viaja un secreto, más chances hay de que se filtre en un log, un proxy o un error.

## Intercambio: credencial larga por token corto

Es el mismo patrón que access y refresh token, pero para máquinas:

```
1. POST /auth/token
   { "client_id": "integracion-erp", "client_secret": "<app key>" }

2. <- { "access_token": "<JWT de 15 min>", "expires_in": 900 }

3. GET /api/v1/pedidos
   Authorization: Bearer <JWT>
   (durante 15 minutos, sin volver a mandar la app key)
```

Detalles:

* La **app key** se guarda **hasheada**, como un refresh token. Se muestra una sola vez al crearla.
* El JWT de servicio tiene un `sub` que identifica a la **integración**, no a una persona, y lleva **scopes** acotados:

```json
{ "sub": "integracion-erp", "empresa": "e-42", "scope": "pedidos:leer", "exp": 1760280000 }
```

* La API puede distinguir tokens de usuario y de servicio (por ejemplo, con un claim o una audiencia propia) y declarar qué tipo acepta cada ruta.
* Revocar una integración es revocar su app key: el JWT que ya tiene muere en minutos.

## Rate limiting: el problema

Sin límites:

* Alguien prueba 10.000 contraseñas por minuto contra `/login`.
* Una integración con un bug hace un bucle infinito y satura la base.
* Un scraper descarga todo tu catálogo.

## Algoritmos

| Algoritmo | Cómo funciona | Contra |
| --- | --- | --- |
| Ventana fija | "Máximo 100 por minuto", el contador se reinicia cada minuto | Permite 200 en 2 segundos, justo en el borde entre dos minutos |
| Ventana deslizante | Cuenta las peticiones de los últimos 60 segundos | Más memoria |
| Token bucket | Un balde de N fichas que se rellena a ritmo constante; cada petición gasta una | Permite ráfagas cortas controladas: suele ser el más práctico |

## En Ktor

Ktor trae un plugin de rate limiting:

```kotlin
install(RateLimit) {
  register(RateLimitName("login")) {
    rateLimiter(limit = 5, refillPeriod = 1.minutes)
    requestKey { call -> call.request.origin.remoteAddress }
  }
  register(RateLimitName("api")) {
    rateLimiter(limit = 600, refillPeriod = 1.minutes)
    requestKey { call -> call.principal<JWTPrincipal>()?.subject ?: "anonimo" }
  }
}

routing {
  rateLimit(RateLimitName("login")) {
    post("/auth/login") { /* ... */ }
  }
}
```

* `requestKey` decide **a quién** se le cuenta: por IP para el login, por usuario o integración para la API.
* Al pasarse del límite, Ktor responde `429 Too Many Requests` con el header `Retry-After`.

## Límites por instancia vs límites globales

El plugin de Ktor cuenta **en memoria**. Con 3 instancias detrás de un balanceador, el límite real es el triple. Para un límite global, el contador tiene que vivir en un lugar compartido, típicamente **Redis** ([Redis](../17-resiliencia-e-integraciones/04-redis.md)).

## Qué limitar y cómo

* **Login y recuperación de contraseña:** límites estrictos, por IP **y** por cuenta (un atacante puede usar muchas IP contra una sola cuenta).
* **API general:** por usuario o integración, con límites generosos.
* **Endpoints caros** (reportes, exportaciones): límites propios, más bajos.
* Detrás de un balanceador, la IP real del cliente viene en un header (`X-Forwarded-For`). Confía en él **solo** si lo pone tu propio proxy; si no, cualquiera lo falsifica.

-----

## Resumen en 5 líneas

1. Los sistemas se autentican con una credencial propia (app key o client secret) que se guarda hasheada.
2. La credencial larga se intercambia por un JWT corto con scopes, así el secreto viaja lo mínimo.
3. Rate limiting protege contra fuerza bruta, bugs en bucle y abuso; el token bucket permite ráfagas controladas.
4. En Ktor, `install(RateLimit)` con una clave por IP, usuario o integración, y responde `429` con `Retry-After`.
5. El límite en memoria es por instancia; para un límite global, usa un contador compartido en Redis.

-----

## Siguiente lección

[7. Cifrado con AES-GCM](07-cifrado-aes-gcm.md): cuando sí necesitas recuperar el secreto original.
