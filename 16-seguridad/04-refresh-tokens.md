# Refresh tokens: rotación, revocación y logout

## En una frase

El access token dura minutos y no se puede revocar; el refresh token dura días, se guarda en la base y sirve para pedir access tokens nuevos. Cada vez que se usa, se **rota**, y si alguien reutiliza uno viejo, se asume que lo robaron y se revoca toda la sesión.

-----

## Antes de empezar

Conviene que ya sepas:

* Qué es un JWT y por qué dura poco: [JWT con RS256](02-jwt-rs256.md).
* Compare-and-swap con `UPDATE ... WHERE`: [Bloqueos y concurrencia](../15-persistencia/06-bloqueos-y-concurrencia.md).

Palabras nuevas (también están en el [Glosario](Glosario.md)):

* **Access token:** Token de vida corta que se manda en cada petición.
* **Refresh token:** Token de vida larga que solo sirve para obtener access tokens nuevos.
* **Token opaco:** Token sin contenido legible: un valor aleatorio que solo tiene sentido para quien lo guardó.
* **Rotación de refresh tokens:** Emitir un refresh token nuevo en cada uso e invalidar el anterior.
* **Familia de tokens:** La cadena de refresh tokens que nació de un mismo login.
* **Revocación:** Invalidar un token antes de que venza.

-----

## El problema: un JWT no se puede "deshacer"

Un JWT se verifica sin consultar la base. Esa es su virtud y su problema: si lo roban, **es válido hasta que vence**, y no hay forma de anularlo sin volver a consultar algo en cada petición.

Si el access token durara 30 días, un token robado daría 30 días de acceso. Si dura 15 minutos, el daño se limita a 15 minutos. Pero nadie quiere hacer login cada 15 minutos.

La solución es usar **dos** tokens:

| | Access token | Refresh token |
| --- | --- | --- |
| Duración | 5 a 15 minutos | Días o semanas |
| Formato | JWT firmado | Opaco (aleatorio) |
| Se verifica | Con la firma, sin base de datos | Contra la base de datos |
| Se envía | En cada petición | Solo a `/auth/refresh` |
| Revocable | No (espera a que venza) | Sí, borrándolo o marcándolo |

## El refresh token es opaco y se guarda hasheado

```kotlin
fun generarRefreshToken(): String {
  val bytes = ByteArray(32).also { SecureRandom().nextBytes(it) }
  return Base64.getUrlEncoder().withoutPadding().encodeToString(bytes)
}
```

En la base **no** guardas el token, sino su hash. Si roban la tabla, no obtienen tokens usables.

¿Por qué alcanza con SHA-256 y no hace falta Argon2id? Porque el token tiene 256 bits de azar. La fuerza bruta solo sirve contra contraseñas humanas, que son predecibles; contra 32 bytes aleatorios, no.

```sql
CREATE TABLE refresh_tokens (
  id            uuid PRIMARY KEY,
  hash          bytea NOT NULL UNIQUE,
  usuario_id    uuid NOT NULL,
  familia_id    uuid NOT NULL,
  expira_en     timestamptz NOT NULL,
  usado_en      timestamptz,
  revocado_en   timestamptz
);
```

## Rotación: cada uso entrega uno nuevo

```
login         -> access A1 + refresh R1                 (familia F)
refresh(R1)   -> access A2 + refresh R2,  R1 marcado como usado
refresh(R2)   -> access A3 + refresh R3,  R2 marcado como usado
```

Marcar como usado es un **compare-and-swap**, para que dos peticiones simultáneas con el mismo token no ganen las dos:

```sql
UPDATE refresh_tokens
SET usado_en = now()
WHERE hash = :hash
  AND usado_en IS NULL
  AND revocado_en IS NULL
  AND expira_en > now();
```

* 1 fila afectada: el token era válido y ahora quedó consumido. Emites el par nuevo **en la misma transacción**.
* 0 filas afectadas: el token no existe, venció, fue revocado o **ya se había usado**.

## Detección de reutilización

¿Por qué alguien presentaría un refresh token ya usado? Un usuario legítimo siempre usa el último. Si aparece uno viejo, significa que **hay dos copias** del token: la del usuario y la de un atacante.

No sabes cuál es cuál, así que la respuesta segura es **revocar toda la familia**:

```kotlin
sealed interface RefrescarOutcome {
  data class Renovado(val accessToken: String, val refreshToken: String) : RefrescarOutcome
  data object Invalido : RefrescarOutcome
  data object ReutilizacionDetectada : RefrescarOutcome
}
```

```kotlin
val registro = tokens.buscarPorHash(hash) ?: return RefrescarOutcome.Invalido

if (registro.usadoEn != null) {
  tokens.revocarFamilia(registro.familiaId)   // atacante y usuario quedan afuera
  return RefrescarOutcome.ReutilizacionDetectada
}
```

El atacante pierde el acceso. El usuario también, y tiene que volver a hacer login, pero eso es mucho mejor que dejar a un atacante adentro.

**Caso borde:** una app móvil con mala señal manda el refresh, el servidor rota, pero la respuesta nunca llega. La app reintenta con el token viejo y se detecta una "reutilización" falsa. Algunos sistemas dan un margen de pocos segundos en el que el token anterior todavía se acepta. Es un equilibrio entre seguridad y experiencia de usuario.

## Logout y revocación

* **Logout:** revoca el refresh token actual (o toda su familia). El access token sigue vivo hasta que vence, unos minutos como máximo. Por eso dura poco.
* **"Cerrar todas mis sesiones":** revoca todas las familias del usuario.
* **Cambio de contraseña:** revoca todas las familias. Si alguien tenía una sesión robada, queda afuera.

Si necesitas cortar el access token **en el acto** (por ejemplo, al bloquear una cuenta), puedes mantener una lista negra de `jti` revocados hasta su `exp`. Pero eso vuelve a consultar algo en cada petición, así que úsalo solo cuando hace falta.

## Dónde guarda los tokens el cliente

| Cliente | Dónde |
| --- | --- |
| App Android | Almacenamiento cifrado respaldado por el Android Keystore |
| Web | El refresh token en una cookie `HttpOnly`, `Secure`, `SameSite`; nunca en `localStorage` |

-----

## Resumen en 5 líneas

1. El access token es un JWT corto que no se puede revocar; el refresh token es largo, opaco y vive en la base.
2. El refresh token se genera con `SecureRandom` y se guarda hasheado (SHA-256 alcanza por su alta entropía).
3. Cada uso rota el token con un compare-and-swap: solo una petición puede consumirlo.
4. Si se presenta un token ya usado, hay dos copias: se revoca toda la familia.
5. Logout, cerrar sesiones y cambiar la contraseña revocan refresh tokens; el access token muere solo en minutos.

-----

## Siguiente lección

[5. RBAC y multi-tenancy](05-rbac-y-multitenancy.md): ya sabes quién es; ahora, qué puede hacer.
