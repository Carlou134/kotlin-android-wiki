# Almacenamiento S3 y MinIO

## En una frase

Los archivos (fotos, PDFs, mapas) no van en la base de datos: van en un almacenamiento de objetos compatible con S3, y la base guarda solo la **key** y los metadatos; en desarrollo local, MinIO hace de S3.

-----

## Antes de empezar

Conviene que ya sepas:

* Qué es un bucket, una key y una presigned URL: [Presigned URLs](../16-seguridad/08-presigned-urls.md).
* Por qué el código bloqueante va en `Dispatchers.IO`: [Scopes y dispatchers](../10-corrutinas/02-scopes-y-dispatchers.md).

Palabras nuevas (también están en el [Glosario](Glosario.md)):

* **Almacenamiento de objetos:** Sistema que guarda archivos completos (objetos) identificados por una key, sin carpetas reales.
* **Prefijo:** La parte inicial de una key que simula una carpeta, como `empresas/e-42/`.
* **Metadatos:** Datos sobre el archivo: tipo, tamaño, fecha, quién lo subió.
* **Subida multiparte (multipart upload):** Subir un archivo grande en partes, que S3 une al final.
* **Regla de ciclo de vida (lifecycle):** Regla del bucket que borra o mueve objetos automáticamente según su antigüedad.
* **Objeto huérfano:** Un archivo en el bucket que ningún registro de la base referencia.

-----

## ¿Por qué no en la base de datos?

Guardar archivos como `bytea` en PostgreSQL funciona con archivos chicos y pocos. Con miles de fotos:

* La base crece en gigas, y los backups y las restauraciones se vuelven lentos.
* Cada descarga ocupa una conexión del pool mientras transfiere bytes.
* Es mucho más caro por GB que un almacenamiento de objetos.

La división correcta:

| Dónde | Qué |
| --- | --- |
| PostgreSQL | La key, el tipo, el tamaño, quién lo subió, a qué pertenece |
| S3 | Los bytes |

## Conceptos

* **No hay carpetas.** `empresas/e-42/fotos/abc.jpg` es una sola key. Las "carpetas" son prefijos, útiles para listar y para reglas, pero no existen como tales.
* **Los objetos se reemplazan enteros.** No se edita la mitad de un archivo: se sube de nuevo.
* **La API de S3 es un estándar de hecho.** AWS S3, MinIO, Cloudflare R2 y otros la implementan. El mismo código funciona contra todos cambiando la URL.

## El cliente

Con el SDK de AWS para Java (v2):

```kotlin
val s3 = S3Client.builder()
  .region(Region.US_EAST_1)
  .endpointOverride(URI(config.endpoint))   // http://localhost:9000 en local con MinIO
  .credentialsProvider(
    StaticCredentialsProvider.create(AwsBasicCredentials.create(config.accessKey, config.secretKey))
  )
  .forcePathStyle(true)                     // MinIO necesita URLs del estilo /bucket/key
  .build()
```

Las operaciones básicas:

```kotlin
// Subir
s3.putObject(
  PutObjectRequest.builder().bucket(bucket).key(key).contentType("application/pdf").build(),
  RequestBody.fromBytes(bytes),
)

// Bajar
val bytes = s3.getObjectAsBytes(GetObjectRequest.builder().bucket(bucket).key(key).build()).asByteArray()

// ¿Existe? (lanza NoSuchKeyException si no)
s3.headObject(HeadObjectRequest.builder().bucket(bucket).key(key).build())

// Borrar
s3.deleteObject(DeleteObjectRequest.builder().bucket(bucket).key(key).build())
```

Este cliente es **bloqueante**: envuelve las llamadas en `withContext(Dispatchers.IO)` dentro del adapter. El SDK también tiene un cliente asíncrono, si lo necesitas.

## Diseñar las keys

```
empresas/{empresaId}/reportes/{año}/{mes}/{uuid}.pdf
```

* **El tenant primero:** permite listar, medir y borrar todo lo de una empresa por prefijo.
* **Un UUID, no el nombre original:** el nombre que eligió el usuario puede tener caracteres raros, repetirse o revelar información. Guárdalo en la base como metadato.
* **La key la genera el backend**, nunca el cliente (lo viste en [Presigned URLs](../16-seguridad/08-presigned-urls.md)).

## Base y bucket coherentes

Subir un archivo y registrarlo son **dos sistemas**: no hay una transacción que abarque los dos. ¿Qué pasa si uno falla?

| Orden | Si falla el segundo paso | Resultado |
| --- | --- | --- |
| Registrar en la base → subir | La base apunta a un archivo que no existe | Error al descargar |
| Subir → registrar en la base | El archivo queda sin registro | Objeto huérfano: ocupa espacio, nadie lo ve |

El segundo orden es el preferido: un huérfano **no rompe nada**, solo ocupa espacio. Y se limpia con un job periódico que borra los objetos sin registro con más de un día de antigüedad.

Para borrar, al revés: primero se marca o se borra en la base (el usuario deja de verlo en el acto) y después se borra en S3. Si el borrado en S3 falla, queda un huérfano más para el job de limpieza.

## Archivos grandes

* Para archivos de más de unos 100 MB, usa **subida multiparte**: el archivo se sube en partes, que se pueden reintentar por separado. Desde el cliente, con presigned URLs por parte.
* Nunca cargues un archivo grande entero en memoria (`ByteArray`): usa streams.

## Reglas de ciclo de vida

El bucket puede borrar objetos solo:

* Exportaciones temporales: borrar después de 7 días.
* Subidas multiparte que nunca se completaron: limpiar después de 1 día.

Es más confiable que un job propio, porque lo hace el almacenamiento.

## En local: MinIO

MinIO corre en un contenedor y expone la API de S3 (puerto 9000) y una consola web (puerto 9001). Un contenedor auxiliar crea los buckets al arrancar. Lo armas en `20-build-y-devops`, con docker-compose.

## Detrás de un puerto

```kotlin
interface AlmacenamientoArchivos {
  suspend fun subir(key: ObjectKey, contenido: ByteArray, tipo: String)
  suspend fun descargar(key: ObjectKey): ByteArray?
  suspend fun borrar(key: ObjectKey)
  fun urlDescarga(key: ObjectKey, duracion: Duration): URL
}
```

En los tests de use cases, una implementación en memoria (`Map<ObjectKey, ByteArray>`). En los tests de integración del adapter, MinIO con Testcontainers.

-----

## Resumen en 5 líneas

1. Los bytes van en S3; la base guarda la key y los metadatos.
2. No hay carpetas, solo prefijos; la API de S3 es un estándar que MinIO implementa para desarrollo local.
3. Las keys empiezan por el tenant, usan un UUID y las genera el backend.
4. Sube primero y registra después: un huérfano no rompe nada y se limpia con un job.
5. Archivos grandes con subida multiparte y streams; reglas de ciclo de vida para lo temporal.

-----

## Siguiente lección

[6. Correo SMTP](06-correo-smtp.md): enviar emails sin duplicarlos.
