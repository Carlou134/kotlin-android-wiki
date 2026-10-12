# Glosario

Las palabras que aparecen en las lecciones de esta carpeta, explicadas de forma simple. Están en orden alfabético.

-----

**Alerta accionable.** Alerta que, cuando suena, requiere que alguien haga algo concreto.

**Appender.** El destino de los logs en Logback: la consola, un archivo.

**Cardinalidad.** Cantidad de combinaciones distintas de valores de tags de una métrica. Alta cardinalidad = muchas series = problemas.

**Correlation ID.** Identificador que une todos los logs de una misma petición o ejecución.

**Counter.** Métrica que solo sube: cantidad de peticiones, de errores.

**Dato personal (PII).** Información que identifica a una persona: email, nombre, documento, teléfono.

**Encoder.** El formato en que un appender de Logback escribe cada log: texto o JSON.

**Gauge.** Métrica que sube y baja: conexiones en uso, tamaño de una cola.

**Health check.** Endpoint que indica si la app está viva (liveness) o lista para recibir tráfico (readiness).

**kotlin-logging.** Capa fina sobre SLF4J con una API idiomática para Kotlin.

**Log.** Registro de algo que pasó en la aplicación.

**Log estructurado.** Log escrito como objeto (normalmente JSON) con campos separados, en lugar de una frase libre.

**Logback.** Implementación de SLF4J que decide qué se escribe, dónde y en qué formato.

**MDC (Mapped Diagnostic Context).** Mapa de valores que Logback agrega automáticamente a cada log del hilo actual.

**`MDCContext`.** Elemento de contexto de corrutina que lleva el MDC con la corrutina aunque cambie de hilo.

**Métrica.** Valor numérico medido a lo largo del tiempo.

**Método RED.** Medir de cada servicio la tasa de peticiones (Rate), los errores (Errors) y la duración (Duration).

**Micrometer.** Librería de la JVM para registrar métricas, independiente de dónde se envíen.

**Nivel de log.** Qué tan importante es un log: `ERROR`, `WARN`, `INFO`, `DEBUG`, `TRACE`.

**OpenTelemetry.** Estándar abierto para trazas, métricas y logs entre servicios.

**Percentil (p95, p99).** El valor por debajo del cual está el 95 % (o 99 %) de las mediciones.

**Prometheus.** Sistema que recolecta métricas consultando un endpoint de cada app y las guarda como series de tiempo.

**Propagación.** Pasar el correlation ID a los servicios que llamas, para seguir la historia entre sistemas.

**Scrape.** La consulta periódica que hace Prometheus al endpoint `/metrics` de cada instancia.

**SLF4J.** Fachada de logging de la JVM: la API que usa el código, sin atarse a una implementación.

**Tag (label).** Dimensión de una métrica, como `proveedor="a"` o `status="500"`.

**Thread-local.** Valor guardado por hilo: cada hilo ve el suyo.

**Timer.** Métrica que mide duraciones y cuenta cuántas hubo.

**Traza distribuida.** El recorrido completo de una petición por varios servicios, con los tiempos de cada paso.
