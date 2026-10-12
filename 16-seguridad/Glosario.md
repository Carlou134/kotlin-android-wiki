# Glosario

Las palabras que aparecen en las lecciones de esta carpeta, explicadas de forma simple. Están en orden alfabético.

-----

**AAD (Additional Authenticated Data).** Datos que no se cifran pero quedan sellados junto al texto cifrado; sirven para atar el cifrado a su contexto.

**Access token.** Token de vida corta (minutos) que se manda en cada petición.

**AES.** El algoritmo de cifrado simétrico estándar.

**App key.** Credencial de larga vida de una aplicación o integración. Se guarda hasheada.

**Argon2id.** Algoritmo de hash de contraseñas, lento a propósito y que consume mucha memoria.

**Autenticación.** Verificar quién eres. Si falla, `401 Unauthorized`.

**Autorización.** Decidir si puedes hacer algo. Si falla, `403 Forbidden`.

**Bearer token.** Token que da acceso a quien lo porte, sin más pruebas: como dinero en efectivo.

**Bucket.** Contenedor de objetos (archivos) en S3.

**Cifrado.** Transformación de dos sentidos: con la clave, se recupera el original.

**Cifrado simétrico.** La misma clave cifra y descifra.

**Claim.** Cada dato del payload de un JWT, como `sub` o `exp`.

**Client credentials.** Flujo de OAuth 2.0 en el que un servicio se autentica con su propio ID y secreto.

**Entitlement.** Una funcionalidad que una empresa tiene contratada.

**Familia de tokens.** La cadena de refresh tokens que nació de un mismo login.

**Firma asimétrica (RS256).** Una clave privada firma y una clave pública verifica.

**Firma digital.** Prueba matemática de que el contenido no se alteró y de quién lo emitió.

**Firma simétrica (HS256).** La misma clave firma y verifica.

**Fuerza bruta.** Probar millones de valores candidatos hasta que alguno coincida.

**GCM.** Modo de AES que, además de cifrar, autentica: detecta si alguien modificó el texto cifrado.

**Hash.** Función de un solo sentido: del dato sale el hash, pero del hash no se vuelve al dato.

**JWK (JSON Web Key).** Una clave criptográfica representada en JSON.

**JWKS (JWK Set).** Un conjunto de JWK publicado en un endpoint, normalmente `/.well-known/jwks.json`.

**JWT (JSON Web Token).** Token con tres partes en Base64URL: header, payload y firma.

**`kid` (key ID).** Identificador de la clave con que se firmó un JWT; viaja en el header.

**M2M (machine-to-machine).** Comunicación entre sistemas, sin un usuario humano.

**MinIO.** Servidor de almacenamiento compatible con la API de S3, que puedes correr en local.

**Multi-tenancy.** Una sola aplicación que sirve a muchos tenants con sus datos aislados.

**Nonce / IV.** Valor que se usa una sola vez por cifrado y hace que el mismo texto cifre distinto cada vez.

**Object key.** El nombre completo de un archivo dentro de un bucket.

**Permiso.** Una acción concreta, como `pedidos:cancelar`.

**Presigned URL.** URL firmada que autoriza una operación concreta sobre un objeto, por tiempo limitado.

**Rate limiting.** Limitar cuántas peticiones acepta la API por cliente y por tiempo.

**RBAC (Role-Based Access Control).** Control de acceso basado en roles que agrupan permisos.

**Refresh token.** Token de vida larga, opaco y guardado en la base, que solo sirve para obtener access tokens nuevos.

**Revocación.** Invalidar un token antes de que venza.

**Rol.** Un conjunto de permisos con nombre, como `ADMIN` u `OPERADOR`.

**Rotación de claves.** Reemplazar periódicamente una clave por otra nueva, sin cortar el servicio.

**Rotación de refresh tokens.** Emitir un refresh token nuevo en cada uso e invalidar el anterior.

**S3.** Servicio de almacenamiento de objetos de AWS; su API es un estándar de hecho.

**Salt (sal).** Valor aleatorio distinto por usuario que se mezcla con la contraseña antes del hash.

**Scope.** Permiso acotado que lleva un token, como `pedidos:leer`.

**Tag de autenticación.** Sello de 16 bytes que GCM agrega para detectar alteraciones.

**Tenant.** Un cliente del sistema (una empresa) cuyos datos están aislados de los demás.

**Token bucket.** Algoritmo de rate limiting: un balde de fichas que se rellena a ritmo constante; cada petición gasta una.

**Token opaco.** Token sin contenido legible: un valor aleatorio que solo tiene sentido para quien lo guardó.
