# JWKS y rotación de claves

## En una frase

El servicio de autenticación publica sus claves públicas en un endpoint JWKS; cada token dice con qué clave se firmó (`kid`), así la API elige la correcta, y se puede cambiar de clave sin invalidar los tokens vigentes.

-----

## Antes de empezar

Conviene que ya sepas:

* Firmar y verificar un JWT con RS256: [JWT con RS256](02-jwt-rs256.md).

Palabras nuevas (también están en el [Glosario](Glosario.md)):

* **JWK (JSON Web Key):** Una clave criptográfica representada en JSON.
* **JWKS (JWK Set):** Un conjunto de JWK publicado en un endpoint.
* **`kid` (key ID):** Identificador de la clave, que viaja en el header del JWT.
* **Rotación de claves:** Reemplazar periódicamente una clave por otra nueva.

-----

## El problema: ¿cómo consigue la API la clave pública?

Opción ingenua: copiar la clave pública en la configuración de cada API. Funciona hasta que hay que cambiarla. Entonces tienes que actualizar y reiniciar todas las APIs **al mismo tiempo** que el servicio de autenticación. Si no, los tokens nuevos fallan en las APIs viejas.

¿Y por qué cambiarla? Porque las claves se rotan: por política, porque una pudo haberse filtrado, o porque un empleado con acceso se fue.

## JWKS: el llavero público

El servicio de autenticación expone un endpoint público, por convención:

```
GET https://auth.tienda.com/.well-known/jwks.json
```

```json
{
  "keys": [
    { "kty": "RSA", "kid": "k2026-10", "use": "sig", "alg": "RS256", "n": "0vx7...", "e": "AQAB" },
    { "kty": "RSA", "kid": "k2026-07", "use": "sig", "alg": "RS256", "n": "xjlC...", "e": "AQAB" }
  ]
}
```

* `n` y `e` son la clave pública RSA. Publicarlas no tiene riesgo: solo sirven para verificar.
* Puede haber **varias** claves a la vez. Esa es la clave de la rotación.

Con nimbus-jose-jwt, publicar el conjunto es casi una línea:

```kotlin
get("/.well-known/jwks.json") {
  val publicas = JWKSet(clavesActivas.map { it.toPublicJWK() })
  call.respondText(publicas.toString(), ContentType.Application.Json)
}
```

`toPublicJWK()` descarta la parte privada. Verifica dos veces que nunca publiques una clave privada.

## Elegir la clave por `kid`

Cada JWT lleva en el header el `kid` de la clave con que se firmó:

```json
{ "alg": "RS256", "kid": "k2026-10" }
```

La API:

1. Lee el `kid` del header.
2. Busca esa clave en el JWKS (que tiene en caché).
3. Verifica la firma con ella.

En Ktor, el `JwkProvider` hace la búsqueda y el cacheo:

```kotlin
val jwkProvider = JwkProviderBuilder(URI("https://auth.tienda.com/.well-known/jwks.json").toURL())
  .cached(10, 24, TimeUnit.HOURS)        // hasta 10 claves en caché, por 24 horas
  .rateLimited(10, 1, TimeUnit.MINUTES)  // como máximo 10 descargas por minuto
  .build()
```

Si llega un `kid` desconocido, el proveedor vuelve a descargar el JWKS. El `rateLimited` evita que alguien, mandando tokens con `kid` inventados, te haga descargar el JWKS en bucle.

## La rotación paso a paso

El truco es que, durante un tiempo, el JWKS publica la clave nueva **y** la vieja.

| Paso | Qué pasa | Por qué |
| --- | --- | --- |
| 1. Publicar | Se agrega la clave nueva al JWKS, pero todavía no se firma con ella | Las APIs la descargan y la tienen en caché antes de verla en un token |
| 2. Firmar con la nueva | Los tokens nuevos salen con el `kid` nuevo | Los tokens viejos siguen siendo válidos con la clave vieja |
| 3. Esperar | Al menos la duración máxima de un access token | Que venzan todos los tokens firmados con la clave vieja |
| 4. Retirar | Se quita la clave vieja del JWKS | Ya nadie la necesita |

En ningún momento hay un token válido que una API no pueda verificar. Ninguna sesión se corta y no se reinicia nada.

**Rotación de emergencia:** si la clave privada se filtró, no esperes el paso 3. Retírala de inmediato y acepta que los tokens vigentes firmados con ella dejan de funcionar: los usuarios tendrán que renovar el token o volver a iniciar sesión.

## Dónde vive la clave privada

* **Nunca** en el repositorio ni en la imagen de Docker.
* En un gestor de secretos o un KMS, o como mínimo en una variable de entorno o archivo montado al arrancar.
* Solo la tiene el servicio de autenticación.

Por eso tiene sentido que el servicio de autenticación sea una aplicación **separada** de la API principal: un proceso distinto, con sus propios secretos, su propio pool y su propio schema. Si comprometen la API, la clave privada no está ahí.

-----

## Resumen en 5 líneas

1. El JWKS es un endpoint público con las claves públicas del servicio de autenticación.
2. Cada JWT lleva un `kid` en el header, y la API elige la clave correspondiente del JWKS.
3. La API cachea el JWKS y lo vuelve a descargar ante un `kid` desconocido, con un límite de descargas.
4. Rotar es: publicar la nueva, firmar con ella, esperar a que venzan los tokens viejos y retirar la vieja.
5. La clave privada vive solo en el servicio de autenticación, fuera del repositorio, en un gestor de secretos.

-----

## Siguiente lección

[4. Refresh tokens](04-refresh-tokens.md): sesiones largas con access tokens que duran minutos.
