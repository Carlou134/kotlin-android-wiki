# Glosario

Las palabras que aparecen en las lecciones de esta carpeta, explicadas de forma simple. Están en orden alfabético.

-----

**Al menos una vez (at-least-once).** Garantía de que algo ocurre una o más veces; puede haber duplicados.

**Almacenamiento de objetos.** Sistema que guarda archivos completos (objetos) identificados por una key, sin carpetas reales.

**Backoff.** La espera entre un intento y el siguiente.

**Backoff exponencial.** La espera se duplica en cada intento: 1 s, 2 s, 4 s, 8 s...

**Caché.** Copia temporal de un dato caro de obtener, guardada en un lugar rápido.

**Cache-aside.** Patrón en el que la app busca primero en la caché y, si no está, carga de la fuente y la guarda.

**Circuit breaker.** Patrón que deja de llamar a un servicio después de varias fallas seguidas, y falla rápido durante un tiempo.

**Como mucho una vez (at-most-once).** Garantía de que algo ocurre cero o una vez; puede perderse.

**CookieJar.** Componente de OkHttp que guarda y reenvía cookies entre peticiones.

**Decorador.** Clase que implementa la misma interfaz que otra y la envuelve para agregarle comportamiento (por ejemplo, caché).

**Engine (cliente).** La librería HTTP que usa Ktor Client por debajo: OkHttp, CIO, Java, etc.

**Error permanente.** Falla que se va a repetir igual: datos inválidos, credenciales incorrectas.

**Error transitorio.** Falla pasajera que probablemente se resuelva sola: un timeout, un servidor reiniciándose.

**Idempotency key.** Identificador único que el cliente manda con una petición para que el servidor reconozca los duplicados.

**Idempotente.** Que repetirla no cambia el resultado más allá de la primera vez.

**Invalidación.** Borrar o actualizar una entrada de caché cuando el dato original cambia.

**Jakarta Mail.** La API estándar de la JVM para enviar correo (antes JavaMail); Angus Mail es su implementación.

**Jitter.** Azar que se suma a la espera para que muchos clientes no reintenten todos al mismo tiempo.

**Ktor Client.** El cliente HTTP de Ktor, multiplataforma y basado en corrutinas.

**Lettuce.** Cliente de Redis para la JVM, con soporte para corrutinas.

**Mailpit.** Servidor SMTP falso para desarrollo que atrapa los emails y los muestra en una web.

**Metadatos.** Datos sobre un archivo: tipo, tamaño, fecha, quién lo subió.

**`MockEngine`.** Engine falso de Ktor Client que responde lo que tú le digas, para tests.

**Objeto huérfano.** Un archivo en el bucket que ningún registro de la base referencia.

**Outbox.** Patrón que guarda los mensajes a enviar en una tabla, en la misma transacción que el cambio de negocio, para que un job los envíe después.

**Paginación por cursor.** El proveedor devuelve un marcador (`cursor`) para pedir la página siguiente.

**Prefijo.** La parte inicial de una key que simula una carpeta, como `empresas/e-42/`.

**Redis.** Base de datos en memoria de clave y valor, compartida entre instancias.

**Regla de ciclo de vida (lifecycle).** Regla del bucket que borra o mueve objetos automáticamente según su antigüedad.

**SMTP.** El protocolo para enviar correo entre servidores.

**SPF, DKIM y DMARC.** Registros DNS que prueban que tu dominio autorizó el envío de un email.

**STARTTLS.** Mecanismo para cifrar una conexión SMTP que empezó sin cifrar (puerto 587).

**Subida multiparte (multipart upload).** Subir un archivo grande en partes, que S3 une al final.

**Tormenta de reintentos.** Cuando los reintentos de muchos clientes saturan a un servicio que ya estaba con problemas.

**TTL (time to live).** Tiempo después del cual una clave se borra sola.

**Upsert.** Insertar si no existe, actualizar si existe, en una sola operación.
