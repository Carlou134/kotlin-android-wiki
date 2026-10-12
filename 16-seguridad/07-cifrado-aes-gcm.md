# Cifrado simétrico con AES-GCM

## En una frase

Cuando necesitas guardar un secreto y **recuperarlo** después (por ejemplo, la contraseña de la API de un proveedor), lo cifras con AES-GCM: la misma clave cifra y descifra, y el resultado incluye un sello que detecta cualquier alteración.

-----

## Antes de empezar

Conviene que ya sepas:

* La diferencia entre hash y cifrado: [Contraseñas con Argon2id](01-contrasenas-con-argon2id.md).

Palabras nuevas (también están en el [Glosario](Glosario.md)):

* **Cifrado simétrico:** La misma clave cifra y descifra.
* **AES:** El algoritmo de cifrado simétrico estándar.
* **GCM:** Modo de AES que, además de cifrar, autentica: detecta si alguien modificó el texto cifrado.
* **Nonce / IV:** Valor que se usa una sola vez por cifrado y hace que el mismo texto cifre distinto cada vez.
* **Tag de autenticación:** Sello de 16 bytes que GCM agrega para detectar alteraciones.
* **AAD (Additional Authenticated Data):** Datos que no se cifran pero quedan sellados junto al texto cifrado.

-----

## Hash o cifrado: depende de si necesitas el original

| Caso | Necesitas el original | Usa |
| --- | --- | --- |
| Contraseña de un usuario | No: solo comparar | Hash (Argon2id) |
| Refresh token, app key | No: solo comparar | Hash (SHA-256) |
| Usuario y contraseña de la API de un proveedor | **Sí**: hay que mandarlos al proveedor | **Cifrado** (AES-GCM) |

Si tu sistema se conecta en nombre del cliente a la API de un tercero, tiene que guardar esas credenciales y poder leerlas. No se pueden hashear. Pero tampoco se pueden guardar en texto plano.

## Por qué GCM

Cifrar oculta el contenido, pero no impide que alguien con acceso a la base **modifique** los bytes cifrados. Con modos viejos (como CBC), un atacante puede alterar el cifrado y producir cambios controlados en el texto descifrado.

GCM es **cifrado autenticado**: agrega un **tag** de 16 bytes. Al descifrar, si un solo bit cambió, falla con una excepción en vez de devolver basura. Cifrar sin autenticar es, hoy, un error de diseño.

## El nonce: la regla que no se rompe

Cada cifrado necesita un nonce de 12 bytes. La regla:

> **Nunca reutilices un nonce con la misma clave.**

Si dos mensajes se cifran con la misma clave y el mismo nonce, un atacante puede obtener el XOR de ambos textos y además **falsificar** tags. No es una debilidad teórica: rompe GCM por completo.

La forma práctica de cumplirla es generar el nonce **al azar** con `SecureRandom` en cada cifrado. Con 12 bytes aleatorios, es seguro para miles de millones de cifrados por clave, de sobra para guardar credenciales.

## Cifrar y descifrar

```kotlin
class AesGcmCifrador(private val clave: SecretKey) {

  private val random = SecureRandom()

  fun cifrar(textoPlano: ByteArray, aad: ByteArray): ByteArray {
    val nonce = ByteArray(12).also { random.nextBytes(it) }
    val cipher = Cipher.getInstance("AES/GCM/NoPadding")
    cipher.init(Cipher.ENCRYPT_MODE, clave, GCMParameterSpec(128, nonce))
    cipher.updateAAD(aad)
    val cifrado = cipher.doFinal(textoPlano) // incluye el tag al final
    return nonce + cifrado
  }

  fun descifrar(datos: ByteArray, aad: ByteArray): ByteArray {
    val nonce = datos.copyOfRange(0, 12)
    val cifrado = datos.copyOfRange(12, datos.size)
    val cipher = Cipher.getInstance("AES/GCM/NoPadding")
    cipher.init(Cipher.DECRYPT_MODE, clave, GCMParameterSpec(128, nonce))
    cipher.updateAAD(aad)
    return cipher.doFinal(cifrado) // lanza AEADBadTagException si algo cambió
  }
}
```

* El nonce no es secreto: se guarda pegado al cifrado (`nonce + cifrado + tag`).
* `GCMParameterSpec(128, ...)` pide un tag de 128 bits (16 bytes).
* La clave es de 32 bytes, es decir AES-256: `SecretKeySpec(bytes, "AES")`.

## AAD: atar el cifrado a su contexto

Sin AAD, un atacante con acceso a la base puede **copiar** las credenciales cifradas de la empresa A a la fila de la empresa B. Se descifran perfecto, porque la clave es la misma.

Con AAD, el cifrado queda sellado junto a su contexto:

```kotlin
val aad = "empresa=${empresaId.value};integracion=${integracionId.value}".toByteArray()
val cifrado = cifrador.cifrar(credenciales, aad)
```

Si alguien mueve los bytes a otra fila, el AAD de esa fila es distinto y el descifrado falla. El AAD no se guarda cifrado: se reconstruye a partir de la fila.

## Manejo de claves

El cifrado es tan seguro como el lugar donde guardas la clave:

* **Nunca** en el código, en el repositorio ni en la misma base de datos que los datos cifrados.
* En un gestor de secretos, un KMS, o al menos una variable de entorno inyectada al arrancar.
* **Versionada:** guarda junto al cifrado un identificador de la clave (por ejemplo, un byte de versión al inicio). Así, cuando rotas la clave, sabes con cuál descifrar cada registro, y puedes recifrar los viejos de a poco.

A mayor escala se usa **envelope encryption**: cada registro se cifra con su propia clave de datos, y esa clave se cifra con una clave maestra que vive en un KMS y nunca sale de él.

## Detrás de un puerto

```kotlin
interface CifradorDeSecretos {
  fun cifrar(secreto: String, contexto: ContextoCifrado): ByteArray
  fun descifrar(datos: ByteArray, contexto: ContextoCifrado): String
}
```

El dominio guarda y recupera credenciales sin saber nada de AES. La configuración (cargar la clave, armar el cifrador) es un módulo de DI más en el composition root.

-----

## Resumen en 5 líneas

1. Si necesitas recuperar el secreto, se cifra; si solo necesitas compararlo, se hashea.
2. AES-GCM cifra y autentica: si alguien modifica un bit, el descifrado falla.
3. Nonce de 12 bytes aleatorio y único por cifrado; reutilizarlo con la misma clave rompe GCM.
4. El AAD ata el cifrado a su contexto (empresa, registro), para que no se pueda copiar a otra fila.
5. La clave vive fuera del código y de la base, versionada para poder rotarla.

-----

## Siguiente lección

[8. Presigned URLs](08-presigned-urls.md): dar acceso a archivos sin regalar las credenciales.
