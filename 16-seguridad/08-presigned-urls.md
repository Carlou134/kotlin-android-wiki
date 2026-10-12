# Presigned URLs

## En una frase

Una presigned URL es un link a un archivo en S3 (o MinIO), firmado por el backend con sus credenciales, que permite **una** operación sobre **un** archivo durante **unos minutos**, sin que el cliente vea las credenciales y sin que el archivo pase por tu servidor.

-----

## Antes de empezar

Conviene que ya sepas:

* Qué es una firma y qué es un bearer token: [JWT con RS256](02-jwt-rs256.md).
* Autorización por empresa: [RBAC y multi-tenancy](05-rbac-y-multitenancy.md).

Palabras nuevas (también están en el [Glosario](Glosario.md)):

* **S3:** Servicio de almacenamiento de objetos de AWS; su API es un estándar de hecho.
* **MinIO:** Servidor de almacenamiento compatible con la API de S3, que puedes correr en local.
* **Bucket:** Contenedor de objetos (archivos) en S3.
* **Object key:** El "nombre completo" de un archivo dentro de un bucket, como `empresas/e-42/fotos/abc.jpg`.
* **Presigned URL:** URL firmada que autoriza una operación concreta sobre un objeto, por tiempo limitado.

-----

## El problema

Los usuarios suben fotos y descargan reportes PDF. Los archivos viven en un bucket privado. ¿Cómo llegan al usuario?

| Opción | Problema |
| --- | --- |
| Bucket público | Cualquiera con el link (o que lo adivine) ve archivos de cualquier empresa |
| Darle las credenciales de S3 al cliente | Con ellas puede leer y borrar **todo** el bucket |
| El backend hace de intermediario (*proxy*) | Cada archivo de 50 MB pasa por tu servidor: memoria, ancho de banda e hilos ocupados |

## La solución

El backend **autoriza** y **firma**; el archivo viaja directo entre el cliente y S3.

```
Descarga:
1. Cliente -> Backend:  GET /api/v1/reportes/r-7/descarga
2. Backend: ¿el usuario puede ver r-7? (RBAC, tenant)
3. Backend firma una URL:  GET del objeto "empresas/e-42/reportes/r-7.pdf", vence en 5 min
4. Backend -> Cliente:  { "url": "https://s3.../r-7.pdf?X-Amz-Signature=...&X-Amz-Expires=300" }
5. Cliente -> S3:  GET <url>   (S3 verifica la firma y el vencimiento)
```

La firma cubre el método, el bucket, la key y el vencimiento. Si el cliente cambia **cualquiera** de esos datos en la URL, la firma deja de coincidir y S3 la rechaza.

## Generarla

Con el SDK de AWS para Java (v2), que también funciona con MinIO:

```kotlin
val presigner = S3Presigner.builder()
  .region(Region.US_EAST_1)
  .endpointOverride(URI(config.s3Endpoint))          // http://localhost:9000 para MinIO
  .credentialsProvider(credenciales)
  .serviceConfiguration(
    S3Configuration.builder().pathStyleAccessEnabled(true).build() // MinIO lo necesita
  )
  .build()

fun urlDescarga(key: String, duracion: Duration): URL {
  val solicitud = GetObjectPresignRequest.builder()
    .signatureDuration(duracion.toJavaDuration())
    .getObjectRequest { it.bucket(config.bucket).key(key) }
    .build()
  return presigner.presignGetObject(solicitud).url()
}
```

Generar la URL **no** hace ninguna llamada de red: es un cálculo local con las credenciales. Es barato.

## Subidas

Para subir, el flujo es parecido, con una presigned URL de `PUT`:

```
1. Cliente -> Backend:  POST /api/v1/fotos  { "tipo": "image/jpeg", "tamano": 2400000 }
2. Backend: autoriza, valida el tipo y el tamaño, y GENERA la key: empresas/e-42/fotos/<uuid>.jpg
3. Backend -> Cliente:  { "uploadUrl": "<PUT firmado, 10 min>", "fotoId": "f-9" }
4. Cliente -> S3:  PUT <uploadUrl>  (el archivo)
5. Cliente -> Backend:  POST /api/v1/fotos/f-9/confirmar
6. Backend: verifica que el objeto existe en S3 (HEAD) y lo marca como disponible
```

Detalles importantes:

* **La key la decide el backend**, nunca el cliente. Si el cliente elige la key, puede pisar archivos de otros o escribir en la carpeta de otra empresa.
* Incluye el `Content-Type` en la firma, así el cliente no puede subir un ejecutable diciendo que es una foto.
* Una URL de `PUT` simple **no** limita el tamaño. Si necesitas un máximo, usa *presigned POST* con una política de tamaño, o verifica el tamaño con `HEAD` en la confirmación y borra lo que no cumpla.

## Riesgos

Una presigned URL es un **bearer**: quien la tenga puede usarla hasta que vence, sin autenticarse.

* **Vencimientos cortos:** minutos, no días.
* **No la registres en logs** ni la guardes en la base: genera una nueva cada vez que se pide.
* **Autoriza antes de firmar:** la URL hereda la decisión de autorización del momento en que se firmó.
* **Keys no adivinables:** usa UUID en las keys. La firma ya protege, pero no conviene exponer nombres de archivo con datos.

## Detrás de un puerto

```kotlin
interface AlmacenamientoArchivos {
  fun urlDescarga(key: ObjectKey, duracion: Duration): URL
  fun urlSubida(key: ObjectKey, tipo: String, duracion: Duration): URL
  suspend fun existe(key: ObjectKey): Boolean
}
```

El use case decide **quién** puede acceder y **qué** key corresponde; el adapter de S3 solo firma. En desarrollo local, el mismo adapter apunta a MinIO (lo verás en `20-build-y-devops`, con docker-compose).

-----

## Resumen en 5 líneas

1. Una presigned URL autoriza una operación sobre un objeto, por tiempo limitado, sin exponer credenciales.
2. El backend autoriza y firma; el archivo viaja directo entre el cliente y S3, sin pasar por el servidor.
3. La firma cubre método, bucket, key y vencimiento: cambiar cualquiera la invalida.
4. En las subidas, la key la genera el backend, se firma el `Content-Type` y se confirma con `HEAD`.
5. Es un bearer: vencimientos cortos, nada de logs y autorización antes de firmar.

-----

## Siguiente lección

Terminaste la seguridad. Vuelve al índice: [README principal](../README.md).
