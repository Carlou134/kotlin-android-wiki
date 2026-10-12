# Correo SMTP

## En una frase

Un backend envía emails hablando con un servidor SMTP (con Jakarta Mail en la JVM); como un email no se puede deshacer, se envía **fuera** de las transacciones, se registra para no duplicarlo, y en desarrollo se usa un servidor falso como Mailpit para ver los correos sin mandarlos de verdad.

-----

## Antes de empezar

Conviene que ya sepas:

* Al menos una vez vs como mucho una vez: [Idempotencia](02-idempotencia.md).
* Jobs programados: [Jobs programados](../10-corrutinas/04-jobs-programados.md).

Palabras nuevas (también están en el [Glosario](Glosario.md)):

* **SMTP:** El protocolo para enviar correo entre servidores.
* **STARTTLS:** Mecanismo para cifrar una conexión SMTP que empezó sin cifrar (puerto 587).
* **Jakarta Mail:** La API estándar de la JVM para enviar correo (antes JavaMail); Angus Mail es su implementación.
* **Mailpit:** Servidor SMTP falso para desarrollo que atrapa los emails y los muestra en una web.
* **SPF, DKIM y DMARC:** Registros DNS que prueban que tu dominio autorizó el envío, para que el email no caiga en spam.

-----

## Cómo funciona

Tu backend no entrega el email al destinatario: se lo entrega a un **servidor SMTP** (el de tu proveedor de correo), y ese servidor se encarga del resto.

```
Backend ──SMTP──> servidor del proveedor ──> servidor del destinatario ──> bandeja de entrada
```

Puertos habituales:

| Puerto | Seguridad |
| --- | --- |
| 587 | STARTTLS: empieza sin cifrar y cambia a TLS (lo más común) |
| 465 | TLS desde el inicio |
| 25 | Entre servidores; normalmente bloqueado para aplicaciones |
| 1025 | Mailpit en local, sin cifrado ni autenticación |

## Enviar con Jakarta Mail

```kotlin
class SmtpNotificador(private val config: SmtpConfig) : NotificadorEmail {

  private val sesion: Session = Session.getInstance(
    Properties().apply {
      put("mail.smtp.host", config.host)
      put("mail.smtp.port", config.puerto.toString())
      put("mail.smtp.auth", config.usuario != null)
      put("mail.smtp.starttls.enable", config.starttls)
      put("mail.smtp.connectiontimeout", "5000")   // ms para conectar
      put("mail.smtp.timeout", "10000")            // ms para leer
    },
    config.usuario?.let { usuario ->
      object : Authenticator() {
        override fun getPasswordAuthentication() = PasswordAuthentication(usuario, config.password)
      }
    },
  )

  override suspend fun enviar(email: Email) = withContext(Dispatchers.IO) {
    val mensaje = MimeMessage(sesion).apply {
      setFrom(InternetAddress(config.remitente))
      setRecipients(Message.RecipientType.TO, InternetAddress.parse(email.para))
      subject = email.asunto
      setText(email.cuerpoTexto, "UTF-8")
    }
    Transport.send(mensaje)
  }
}
```

Puntos clave:

* **`Transport.send` es bloqueante:** va en `Dispatchers.IO`.
* **Timeouts explícitos:** por defecto, Jakarta Mail puede esperar **para siempre** a un servidor que no responde.
* **La sesión se crea una vez** y se reutiliza.
* La configuración (host, puerto, credenciales) viene de variables de entorno: Mailpit en local, el proveedor real en producción, **sin cambiar código**.

Para emails con HTML, se arma un `MimeMultipart("alternative")` con una parte de texto y una HTML. Incluir la versión de texto mejora la entrega y la accesibilidad.

## En local: Mailpit

Mailpit es un servidor SMTP que **no envía nada**: atrapa cada email y lo muestra en una interfaz web.

* SMTP en el puerto `1025`, sin autenticación.
* Interfaz web en `http://localhost:8025`.

Puedes probar todo el flujo de alertas o de recuperación de contraseña sin riesgo de mandar un email a un cliente real. Mailpit también tiene una API HTTP, útil para verificar en tests de integración que el email salió. Se levanta con docker-compose ([Docker y docker-compose](../20-build-y-devops/03-docker-y-docker-compose.md)).

## Nunca dentro de una transacción

```kotlin
transaction {
  pedidos.marcarEnviado(pedidoId)
  notificador.enviar(emailDeEnvio)   // ❌
}
```

Dos problemas:

1. Si el email se envía y después la transacción hace `ROLLBACK`, el cliente recibe un aviso de algo que **no pasó**.
2. Si el servidor SMTP tarda 10 segundos, la transacción retiene la conexión y los locks 10 segundos.

El email va **después** del `COMMIT`. O mejor, en la misma transacción se registra que hay que enviarlo (patrón outbox), y un job lo envía.

## Sin duplicados

Un job revisa cada 5 minutos qué dispositivos dejaron de reportar y manda alertas. Sin cuidado, manda la **misma** alerta cada 5 minutos, o dos instancias la mandan a la vez.

La solución es registrar cada notificación con una clave única **antes** de enviar:

```sql
CREATE TABLE notificaciones_enviadas (
  alerta_id    uuid NOT NULL,
  destinatario text NOT NULL,
  enviada_en   timestamptz NOT NULL DEFAULT now(),
  PRIMARY KEY (alerta_id, destinatario)
);
```

```kotlin
val esNueva = notificaciones.registrar(alerta.id, destinatario)   // INSERT; false si ya existía
if (esNueva) {
  notificador.enviar(email)
}
```

* La `PRIMARY KEY` garantiza que, aunque dos instancias lo intenten a la vez, **solo una** registra y envía.
* Registrar primero es **como mucho una vez**: si el envío falla después de registrar, el email se pierde. Para que no se pierda, registra con un estado `PENDIENTE`, márcalo `ENVIADA` después del envío, y que un job reintente los pendientes viejos. Lo viste en [Idempotencia](02-idempotencia.md).

## Reintentos

Los errores de SMTP también se clasifican:

* **Transitorios:** no se pudo conectar, timeout, códigos `4xx` del servidor ("intente más tarde"). Se reintentan.
* **Permanentes:** dirección inválida, códigos `5xx` ("buzón inexistente"). No se reintentan; se registran.

## Que el email llegue

Que tu servidor lo envíe no significa que llegue a la bandeja de entrada. Para no caer en spam:

* Envía a través de un proveedor de correo transaccional, no desde tu propio servidor.
* Configura en el DNS de tu dominio los registros **SPF**, **DKIM** y **DMARC**, que prueban que tu dominio autorizó el envío.
* El remitente debe ser de tu dominio, y coherente con esos registros.

## Detrás de un puerto

```kotlin
interface NotificadorEmail {
  suspend fun enviar(email: Email)
}
```

El use case decide **qué** se notifica y a **quién**; el adapter decide **cómo**. Mañana puedes cambiar SMTP por la API HTTP de un proveedor sin tocar el negocio.

-----

## Resumen en 5 líneas

1. El backend entrega el email a un servidor SMTP (puerto 587 con STARTTLS), con Jakarta Mail en `Dispatchers.IO` y timeouts explícitos.
2. En local, Mailpit atrapa todos los emails y los muestra en `localhost:8025`; solo cambia la configuración.
3. Nunca envíes un email dentro de una transacción: después del `COMMIT`, o con un outbox y un job.
4. Registra cada notificación con una clave única para no duplicarla, y con estados si no puedes permitirte perderla.
5. Para que llegue, usa un proveedor transaccional y configura SPF, DKIM y DMARC.

-----

## Siguiente lección

Terminaste resiliencia e integraciones. Vuelve al índice: [README principal](../README.md).
