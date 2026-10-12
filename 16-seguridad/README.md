# Seguridad e identidad

En esta carpeta aprendes a **proteger un backend**: cómo guardar contraseñas, cómo emitir y verificar tokens que prueban quién es el usuario, cómo rotar claves y tokens sin cortar sesiones, cómo decidir qué puede hacer cada uno dentro de cada empresa, y cómo proteger secretos y archivos.

-----

## Antes de empezar

Conviene que ya hayas leído [Arquitectura](../12-arquitectura/README.md), porque hashing, firma y cifrado se esconden detrás de puertos, y [Persistencia](../15-persistencia/README.md), sobre todo [Bloqueos y concurrencia](../15-persistencia/06-bloqueos-y-concurrencia.md), porque la rotación de refresh tokens es un compare-and-swap.

Si en algún momento aparece una palabra que no conoces, búscala en el [Glosario](Glosario.md).

-----

## Orden de lectura

Las lecciones se apoyan una en la otra, así que conviene leerlas en este orden:

| Lección | Qué aprendes | Qué necesitas saber antes |
| --- | --- | --- |
| [1. Contraseñas con Argon2id](01-contrasenas-con-argon2id.md) | Hashing vs cifrado, salt, por qué un hash lento, parámetros y rehash | Puertos y adapters |
| [2. JWT con RS256](02-jwt-rs256.md) | Estructura de un JWT, claims, firma asimétrica y validación completa | Contraseñas |
| [3. JWKS y rotación de claves](03-jwks-y-rotacion-de-claves.md) | Publicar claves públicas, elegir la clave por `kid` y rotar sin cortar sesiones | JWT |
| [4. Refresh tokens](04-refresh-tokens.md) | Access vs refresh, rotación, detección de reutilización, revocación y logout | JWT y compare-and-swap |
| [5. RBAC y multi-tenancy](05-rbac-y-multitenancy.md) | Roles, permisos, asignaciones por empresa, contexto de autorización y aislamiento de datos | Capa application |
| [6. Tokens de servicio y rate limiting](06-tokens-de-servicio-y-rate-limiting.md) | Autenticación máquina a máquina, intercambio de app keys y límites de peticiones | JWT |
| [7. Cifrado con AES-GCM](07-cifrado-aes-gcm.md) | Cifrado simétrico autenticado, nonce, AAD y manejo de claves | Contraseñas |
| [8. Presigned URLs](08-presigned-urls.md) | Dar acceso temporal a un archivo en S3 sin exponer credenciales | Puertos y adapters |

-----

## El mapa completo en una mirada

```
Contraseñas      ->  "nunca se guardan, se guarda su hash"
   $argon2id$v=19$m=19456,t=2,p=1$<salt>$<hash>

JWT RS256        ->  "un pase firmado que cualquiera verifica, pero solo uno emite"
   header.payload.firma     auth-service firma (privada)  ->  API verifica (pública)

JWKS             ->  "el llavero público"
   GET /.well-known/jwks.json   -> la API elige la clave por "kid"

Refresh tokens   ->  "sesiones largas con pases cortos"
   access 15 min + refresh 30 días; cada uso rota; reutilizar = robo -> revocar la familia

RBAC + tenancy   ->  "quién eres no alcanza: qué puedes hacer y DÓNDE"
   Usuario -> RoleAssignment(empresa, rol) -> Permission
   toda consulta filtra por empresa_id

M2M y rate limit ->  "servicios que se autentican y límites por cliente"
   app key -> token corto     429 Too Many Requests

AES-GCM          ->  "secretos que hay que poder leer de vuelta"
   nonce(12) + cifrado + tag(16), nonce único por mensaje

Presigned URLs   ->  "una llave temporal a un archivo"
   GET https://bucket/archivo?X-Amz-Signature=...&X-Amz-Expires=600
```

-----

## Cómo está armada cada lección

1. **En una frase:** qué es, sin jerga.
2. **Antes de empezar:** qué tienes que saber y las palabras nuevas.
3. **El contenido:** la explicación con ejemplos.
4. **Resumen en 5 líneas:** para repasar.
5. **Siguiente lección:** para seguir el orden.

-----

## Recursos

* [OWASP Password Storage Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Password_Storage_Cheat_Sheet.html)
* [OWASP JSON Web Token Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/JSON_Web_Token_for_Java_Cheat_Sheet.html)
* [RFC 7519: JSON Web Token](https://www.rfc-editor.org/rfc/rfc7519) y [RFC 7517: JSON Web Key](https://www.rfc-editor.org/rfc/rfc7517)
* [OAuth 2.0 Security Best Current Practice (RFC 9700)](https://www.rfc-editor.org/rfc/rfc9700)
* [Documentación de autenticación JWT en Ktor](https://ktor.io/docs/server-jwt.html)

-----

## Después de esta carpeta

Ya sabes quién entra y qué puede hacer. Vuelve al índice: [README principal](../README.md).
