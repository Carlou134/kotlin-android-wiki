# Contraseñas con Argon2id

## En una frase

Las contraseñas **nunca** se guardan: se guarda un hash lento, con sal, calculado con Argon2id, de modo que aunque roben la base de datos sea carísimo recuperar las contraseñas originales.

-----

## Antes de empezar

Conviene que ya sepas:

* Qué es un puerto de salida: [Arquitectura hexagonal](../12-arquitectura/02-arquitectura-hexagonal.md).

Palabras nuevas (también están en el [Glosario](Glosario.md)):

* **Hash:** Función de un solo sentido: de la contraseña sale el hash, pero del hash no se vuelve a la contraseña.
* **Cifrado:** Transformación de dos sentidos: con la clave, se recupera el original.
* **Salt (sal):** Valor aleatorio distinto por usuario que se mezcla con la contraseña antes del hash.
* **Argon2id:** Algoritmo de hash de contraseñas, lento a propósito y que consume mucha memoria.
* **Fuerza bruta:** Probar millones de contraseñas candidatas hasta que alguna coincida.

-----

## El problema: las bases de datos se filtran

Asume que algún día alguien va a leer tu tabla `usuarios`: un backup mal guardado, una inyección SQL, un empleado con demasiado acceso. La pregunta no es *si* pasa, sino *qué obtiene el atacante cuando pasa*.

| Si guardas | El atacante obtiene |
| --- | --- |
| La contraseña en texto plano | Todas las contraseñas, en el acto |
| La contraseña cifrada | Todas, si también roba la clave (que suele estar cerca) |
| `SHA-256(contraseña)` | Casi todas: una GPU calcula miles de millones de SHA-256 por segundo |
| `Argon2id(contraseña, sal)` | Muy pocas: cada intento cuesta tiempo y memoria |

**Hash, no cifrado:** nunca necesitas saber la contraseña original; solo verificar que la que llega es la misma. Para eso alcanza con un camino de ida.

## Por qué un hash *lento*

SHA-256 está diseñado para ser **rápido**, y eso es justo lo malo. Con fuerza bruta, el atacante prueba millones de contraseñas comunes por segundo.

Argon2id está diseñado para ser **caro**:

* **Tiempo:** varias pasadas sobre los datos.
* **Memoria:** cada cálculo necesita decenas de MB. Las GPU tienen mucha potencia de cálculo pero poca memoria por núcleo, así que no pueden paralelizar millones de intentos.

Para tu servidor, un login que tarda 50 ms es invisible. Para el atacante, que necesita miles de millones de intentos, es la diferencia entre horas y siglos.

## Por qué la sal

Sin sal, dos usuarios con la contraseña `123456` tienen el **mismo** hash. El atacante calcula el hash de las contraseñas más comunes una sola vez (tablas precalculadas) y encuentra a todos de golpe.

Con una sal aleatoria por usuario, cada hash es distinto aunque la contraseña sea la misma, y el atacante tiene que atacar **a cada usuario por separado**.

## Parámetros

Argon2id tiene tres parámetros: memoria (`m`), iteraciones (`t`) y paralelismo (`p`). La recomendación de OWASP como **mínimo** es:

```
m = 19 MiB (19456 KiB), t = 2, p = 1
```

Súbelos mientras el login siga siendo aceptable en tu hardware.

El resultado se guarda en un formato estándar que **incluye los parámetros y la sal**:

```
$argon2id$v=19$m=19456,t=2,p=1$c2FsdGFsZWF0b3JpYQ$hashresultante...
```

Así puedes subir los parámetros en el futuro sin romper los hashes viejos: cada hash dice con qué se calculó.

## Detrás de un puerto

La capa application no sabe que existe Argon2id:

```kotlin
interface PasswordHasher {
  fun hashear(contrasena: CharArray): String
  fun verificar(contrasena: CharArray, hashGuardado: String): Boolean
  fun necesitaRehash(hashGuardado: String): Boolean
}
```

La implementación vive en infraestructura. Por ejemplo, con Bouncy Castle, el cálculo del hash es así:

```kotlin
private fun calcular(contrasena: CharArray, sal: ByteArray): ByteArray {
  val parametros = Argon2Parameters.Builder(Argon2Parameters.ARGON2_id)
    .withVersion(Argon2Parameters.ARGON2_VERSION_13)
    .withMemoryAsKB(19_456)
    .withIterations(2)
    .withParallelism(1)
    .withSalt(sal)
    .build()

  val resultado = ByteArray(32)
  Argon2BytesGenerator().apply { init(parametros) }.generateBytes(contrasena, resultado)
  return resultado
}
```

La sal se genera con `SecureRandom` (16 bytes), y al verificar se compara con `MessageDigest.isEqual`, que tarda lo mismo sin importar dónde difieren los bytes. Una comparación normal corta en el primer byte distinto, y medir ese tiempo filtra información.

## Rehash al hacer login

Cuando subes los parámetros, los hashes viejos siguen funcionando. El momento de actualizarlos es el login, el único en que tienes la contraseña en mano:

```kotlin
if (hasher.verificar(contrasena, usuario.hash)) {
  if (hasher.necesitaRehash(usuario.hash)) {
    usuarios.actualizarHash(usuario.id, hasher.hashear(contrasena))
  }
  // continuar con el login
}
```

## No reveles qué usuarios existen

```kotlin
val usuario = usuarios.buscarPorEmail(email)
  ?: return LoginOutcome.CredencialesInvalidas   // ❌ responde en 1 ms
```

Si el email no existe respondes en 1 ms; si existe, tardas 50 ms calculando Argon2id. Midiendo el tiempo, un atacante descubre qué emails están registrados. La solución es calcular un hash igual aunque el usuario no exista, y responder **el mismo** error en los dos casos.

-----

## Resumen en 5 líneas

1. Las contraseñas no se guardan ni se cifran: se guarda un hash de un solo sentido.
2. El hash tiene que ser lento y consumir memoria (Argon2id) para que la fuerza bruta sea carísima.
3. Una sal aleatoria por usuario impide atacar a todos de golpe con tablas precalculadas.
4. El formato `$argon2id$...` guarda parámetros y sal; al hacer login puedes rehashear con parámetros nuevos.
5. Compara en tiempo constante y responde igual (y en el mismo tiempo) si el usuario no existe.

-----

## Siguiente lección

[2. JWT con RS256](02-jwt-rs256.md): qué recibe el usuario después de un login exitoso.
