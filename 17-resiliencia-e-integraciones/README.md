# Resiliencia e integraciones

En esta carpeta aprendes a **hablar con sistemas que no controlas** sin que sus fallas se conviertan en las tuyas: cuándo y cómo reintentar, cómo hacer que repetir una operación no cause efectos dobles, cómo integrar APIs de terceros con Ktor Client, y cómo usar Redis, almacenamiento S3 y correo SMTP.

-----

## Antes de empezar

Conviene que ya hayas leído:

* [Concurrencia estructurada](../10-corrutinas/03-concurrencia-estructurada.md): timeouts y `CancellationException`.
* [Jobs programados](../10-corrutinas/04-jobs-programados.md): donde corren la mayoría de las integraciones.
* [DDD y monolito modular](../12-arquitectura/03-ddd-y-monolito-modular.md): la Anti-Corruption Layer.
* [Bloqueos y concurrencia](../15-persistencia/06-bloqueos-y-concurrencia.md): compare-and-swap y leases.

Si en algún momento aparece una palabra que no conoces, búscala en el [Glosario](Glosario.md).

-----

## Orden de lectura

Las lecciones se apoyan una en la otra, así que conviene leerlas en este orden:

| Lección | Qué aprendes | Qué necesitas saber antes |
| --- | --- | --- |
| [1. Reintentos con backoff](01-reintentos-con-backoff.md) | Errores transitorios vs permanentes, backoff exponencial, jitter y tormentas de reintentos | Corrutinas y timeouts |
| [2. Idempotencia](02-idempotencia.md) | Métodos idempotentes, `Idempotency-Key`, upserts y deduplicación | Reintentos y compare-and-swap |
| [3. Ktor Client y APIs de terceros](03-ktor-client-y-apis-de-terceros.md) | Configurar el cliente, timeouts, autenticación, cookies, paginación, ACL y tests con `MockEngine` | Reintentos, idempotencia y ACL |
| [4. Redis](04-redis.md) | Caché con TTL, cache-aside, contadores y por qué Redis no es la fuente de verdad | Ktor Client |
| [5. Almacenamiento S3 y MinIO](05-almacenamiento-s3-y-minio.md) | Buckets, keys, subir y bajar objetos, y mantener la base y el bucket coherentes | Presigned URLs |
| [6. Correo SMTP](06-correo-smtp.md) | Enviar emails con Jakarta Mail, Mailpit en local, y envíos sin duplicados | Idempotencia |

-----

## El mapa completo en una mirada

```
Reintentos     ->  "fallas pasajeras: reintentar con calma"
   429, 502, 503, 504, timeout -> reintentar      400, 401, 403, 404 -> NO
   espera = random(0, min(tope, base * 2^intento))

Idempotencia   ->  "repetir no causa efectos dobles"
   Idempotency-Key: 7f3c...   ->  misma respuesta, sin duplicar
   INSERT ... ON CONFLICT (id_externo) DO UPDATE

Ktor Client    ->  "un cliente por proveedor, configurado una vez"
   HttpClient(OkHttp) { HttpTimeout; ContentNegotiation; HttpRequestRetry; defaultRequest }
   DTO del proveedor -> toDomain()  (Anti-Corruption Layer)

Redis          ->  "memoria rápida y compartida, con vencimiento"
   GET -> si no está: cargar de la base -> SET ... EX 300

S3 / MinIO     ->  "archivos en un bucket; en la base, solo la key"
   putObject(bucket, "empresas/e-42/fotos/<uuid>.jpg")

SMTP           ->  "emails fuera de la transacción, sin duplicados"
   registrar envío (único) -> Transport.send(mensaje)    Mailpit en local
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

* [AWS Architecture Blog: Exponential Backoff And Jitter](https://aws.amazon.com/blogs/architecture/exponential-backoff-and-jitter/)
* [Stripe: Idempotent requests](https://docs.stripe.com/api/idempotent_requests)
* [Documentación de Ktor Client](https://ktor.io/docs/client-create-new-application.html)
* [Documentación de Redis](https://redis.io/docs/latest/) y [Lettuce](https://redis.github.io/lettuce/)

-----

## Después de esta carpeta

Ya sabes integrarte con el mundo exterior sin que sus fallas te tumben. Vuelve al índice: [README principal](../README.md).
