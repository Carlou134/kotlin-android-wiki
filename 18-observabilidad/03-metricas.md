# Métricas con Micrometer y Prometheus

## En una frase

Una métrica es un **número que se mide en el tiempo** (peticiones por segundo, errores, latencia, conexiones en uso); Micrometer las registra dentro de la app, Prometheus las recolecta cada pocos segundos, y con ellas se arman tableros y alertas que avisan de un problema antes de que lo note un cliente.

-----

## Antes de empezar

Conviene que ya sepas:

* Logs estructurados: [Logging estructurado](01-logging-estructurado.md).
* Plugins de Ktor: [Ktor server y plugins](../14-ktor-y-koin/01-ktor-server-y-plugins.md).
* Jobs y reintentos: [Jobs programados](../10-corrutinas/04-jobs-programados.md) y [Reintentos con backoff](../17-resiliencia-e-integraciones/01-reintentos-con-backoff.md).

Palabras nuevas (también están en el [Glosario](Glosario.md)):

* **Métrica:** Valor numérico medido a lo largo del tiempo.
* **Micrometer:** Librería de la JVM para registrar métricas, independiente de dónde se envíen.
* **Prometheus:** Sistema que recolecta métricas consultando un endpoint de cada app y las guarda como series de tiempo.
* **Counter:** Métrica que solo sube: cantidad de peticiones, de errores.
* **Gauge:** Métrica que sube y baja: conexiones en uso, tamaño de una cola.
* **Timer:** Métrica que mide duraciones y cuenta cuántas hubo.
* **Tag (label):** Dimensión de una métrica, como `proveedor="a"` o `status="500"`.
* **Cardinalidad:** Cantidad de combinaciones distintas de valores de tags.
* **Percentil (p95, p99):** El valor por debajo del cual está el 95 % (o 99 %) de las mediciones.

-----

## Logs, métricas y trazas

| | Logs | Métricas | Trazas |
| --- | --- | --- | --- |
| Responden | ¿Qué pasó exactamente? | ¿Cuánto, cuántas veces, qué tan rápido? | ¿Dónde se fue el tiempo? |
| Volumen | Una línea por evento | Unos pocos números por intervalo | Un registro por paso |
| Ideales para | Investigar un caso | Ver tendencias y alertar | Encontrar cuellos de botella |

Los logs te dicen **por qué** falló una petición. Las métricas te dicen **que** el 8 % de las peticiones están fallando desde las 14:20, aunque nadie haya mirado un solo log.

## Los tres tipos básicos

```kotlin
// Counter: solo sube
registry.counter("sincronizaciones_fallidas", "proveedor", "a").increment()

// Gauge: un valor actual que sube y baja
registry.gauge("alertas_pendientes", colaDeAlertas) { it.size.toDouble() }

// Timer: duración + cantidad
val timer = Timer.builder("sincronizacion_duracion")
  .tag("proveedor", "a")
  .publishPercentiles(0.5, 0.95, 0.99)
  .register(registry)
```

Para medir código `suspend` con un timer:

```kotlin
val muestra = Timer.start(registry)
try {
  sincronizar(proveedor)
} finally {
  muestra.stop(timer)
}
```

## Métricas de Ktor y de la JVM, gratis

```kotlin
val registry = PrometheusMeterRegistry(PrometheusConfig.DEFAULT)

install(MicrometerMetrics) {
  this.registry = registry
  meterBinders = listOf(
    JvmMemoryMetrics(),
    JvmGcMetrics(),
    ProcessorMetrics(),
  )
}

routing {
  get("/metrics") {
    call.respond(registry.scrape())
  }
}
```

> El paquete de `PrometheusMeterRegistry` cambió en versiones recientes de Micrometer; revisa el import según tu versión.

Con esto ya tienes, sin escribir nada más:

* Peticiones HTTP por ruta, método y código de estado, con su duración.
* Memoria, recolección de basura y uso de CPU de la JVM.

HikariCP también publica sus métricas (conexiones activas, en espera, tiempo para obtener una) si le pasas el registry al configurarlo (`metricRegistry`). Son clave para detectar el pool saturado de [PostgreSQL y HikariCP](../15-persistencia/03-postgresql-y-hikaricp.md).

## Cómo funciona Prometheus

Prometheus **pregunta** (*scrape*): cada 15 o 30 segundos hace `GET /metrics` a cada instancia y guarda lo que lee. Lo que ve es texto:

```
# TYPE sincronizaciones_fallidas_total counter
sincronizaciones_fallidas_total{proveedor="a"} 17.0
sincronizaciones_fallidas_total{proveedor="b"} 2.0
```

Con esas series se arman tableros (normalmente en **Grafana**) y reglas de alerta.

`/metrics` no debe estar abierto a internet: muestra rutas, volúmenes y detalles internos. Se expone solo en la red interna, o en un puerto aparte.

## Cardinalidad: el error más caro

Cada combinación distinta de tags es una **serie** que Prometheus guarda por separado.

```kotlin
registry.counter("pedidos_creados", "empresaId", empresaId.value)   // ⚠️ una serie por empresa
registry.counter("pedidos_creados", "usuarioId", usuarioId.value)   // ❌ una serie por usuario
registry.counter("pedidos_creados", "pedidoId", pedidoId.value)     // ❌❌ una serie por pedido
```

Con un tag por usuario o por pedido, la cantidad de series crece sin límite, la memoria de Prometheus explota y las consultas se vuelven lentas. Regla: **los tags tienen pocos valores posibles y conocidos** (proveedor, ruta, código de estado, tipo de resultado). Para "qué pasó con el pedido p-9" están los logs, no las métricas.

Ktor ya agrupa las rutas por su **plantilla** (`/pedidos/{id}`), no por la URL concreta, justamente para evitar este problema.

## Qué medir: el método RED

Para cada servicio o endpoint:

* **R**ate: cuántas peticiones por segundo.
* **E**rrors: cuántas fallan.
* **D**uration: cuánto tardan (en percentiles).

Para cada recurso (pool de conexiones, cola, CPU): cuánto se usa, cuánto está saturado y cuántos errores da.

Y las métricas **de negocio**, que nadie más te va a dar:

* Sincronizaciones por proveedor y resultado (`ok`, `error_transitorio`, `error_permanente`).
* Duración de cada job.
* **Última ejecución exitosa** de cada job (un gauge con el timestamp).
* Alertas enviadas, emails fallidos.

## Promedios mienten, percentiles no

Si 99 peticiones tardan 50 ms y una tarda 10 segundos, el promedio es 150 ms: "todo bien". Pero **uno de cada cien** usuarios espera 10 segundos.

El **p99** (el valor por debajo del cual está el 99 % de las peticiones) muestra esa cola lenta. Mira p95 y p99, no el promedio.

## Alertas que sirven

Una alerta debe ser **accionable**: si suena, alguien tiene que hacer algo. Ejemplos:

| Alerta | Por qué |
| --- | --- |
| Más del 5 % de respuestas `5xx` durante 5 minutos | El servicio está fallando |
| p99 de una ruta por encima de 2 s durante 10 minutos | Algo se está degradando |
| Un job sin ejecución exitosa en 3 intervalos | El job murió o falla siempre, aunque nadie mire sus logs |
| Conexiones del pool en espera por más de 1 minuto | La base o el pool están saturados |

La alerta del job es la más valiosa para un backend con sincronizaciones: detecta el caso en que el job **dejó de correr en silencio**, que ningún log de error va a mostrar, porque no hay ningún log.

Evita las alertas por cosas que se resuelven solas (un `503` aislado de un proveedor que el reintento ya manejó). Si las alertas suenan todo el tiempo, el equipo deja de mirarlas.

## Health checks

Junto a las métricas, la plataforma necesita saber si la app está viva:

* **Liveness** (`/health/live`): "el proceso responde". Si falla, la plataforma reinicia la app.
* **Readiness** (`/health/ready`): "puedo atender peticiones" (por ejemplo, la base responde). Si falla, la plataforma deja de mandarle tráfico, sin reiniciarla.

Son rutas públicas declaradas explícitamente, como viste en [Rutas y autenticación por ruta](../14-ktor-y-koin/03-rutas-y-autenticacion.md).

-----

## Resumen en 5 líneas

1. Las métricas son números en el tiempo: counters (solo suben), gauges (suben y bajan) y timers (duraciones).
2. `MicrometerMetrics` en Ktor da métricas HTTP y de la JVM; Prometheus las lee de `/metrics`, que no se expone a internet.
3. Los tags tienen pocos valores conocidos: nunca un ID de usuario o de pedido, o la cardinalidad explota.
4. Mide RED (rate, errores, duración) en percentiles, más métricas de negocio como la última ejecución exitosa de cada job.
5. Las alertas tienen que ser accionables; health checks de liveness y readiness para la plataforma.

-----

## Siguiente lección

Terminaste observabilidad. Vuelve al índice: [README principal](../README.md).
