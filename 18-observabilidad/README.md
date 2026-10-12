# Observabilidad

En esta carpeta aprendes a **saber qué está pasando en producción** sin poder poner un breakpoint: logs estructurados que se pueden buscar, un identificador que une todos los logs de una misma petición o ejecución, y métricas que te avisan de un problema antes de que te lo cuente un cliente.

-----

## Antes de empezar

Conviene que ya hayas leído [Ktor server y plugins](../14-ktor-y-koin/01-ktor-server-y-plugins.md) (CallId y CallLogging), [Jobs programados](../10-corrutinas/04-jobs-programados.md) y [Ktor Client y APIs de terceros](../17-resiliencia-e-integraciones/03-ktor-client-y-apis-de-terceros.md).

Si en algún momento aparece una palabra que no conoces, búscala en el [Glosario](Glosario.md).

-----

## Orden de lectura

Las lecciones se apoyan una en la otra, así que conviene leerlas en este orden:

| Lección | Qué aprendes | Qué necesitas saber antes |
| --- | --- | --- |
| [1. Logging estructurado](01-logging-estructurado.md) | Niveles, SLF4J + Logback + kotlin-logging, logs en JSON, campos y qué nunca loguear | Nada |
| [2. Correlation ID](02-correlation-id.md) | Seguir una petición por todos los logs, MDC con corrutinas, propagación a otros servicios y a los jobs | Logging estructurado y CallId |
| [3. Métricas con Micrometer y Prometheus](03-metricas.md) | Contadores, gauges y timers, métricas de Ktor, cardinalidad, percentiles y alertas | Logging estructurado |

-----

## El mapa completo en una mirada

```
Logs            ->  "qué pasó, con detalle"
   logger.info { "Pedido creado" }  ->  {"level":"INFO","message":"Pedido creado","pedidoId":"p-9","callId":"8f3c"}
   ERROR / WARN / INFO / DEBUG         nunca: contraseñas, tokens, datos personales

Correlation ID  ->  "todos los logs de la misma historia"
   X-Request-Id -> MDC["call-id"] -> cada log, el error al cliente y las llamadas salientes
   withContext(MDCContext()) { ... }    // el MDC sobrevive al cambio de hilo

Métricas        ->  "cuánto, con qué frecuencia, qué tan rápido"
   counter("sincronizaciones_fallidas", "proveedor", "a").increment()
   GET /metrics  <- Prometheus  -> Grafana y alertas
   tags con pocos valores posibles: nunca un userId
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

* [kotlin-logging](https://github.com/oshai/kotlin-logging)
* [logstash-logback-encoder](https://github.com/logfellow/logstash-logback-encoder)
* [Métricas con Micrometer en Ktor](https://ktor.io/docs/server-metrics-micrometer.html)
* [Documentación de Micrometer](https://docs.micrometer.io/micrometer/reference/)
* [Prometheus: buenas prácticas de nombres](https://prometheus.io/docs/practices/naming/)

-----

## Después de esta carpeta

Ya sabes ver tu sistema por dentro. Vuelve al índice: [README principal](../README.md).
