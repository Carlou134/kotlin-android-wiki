# Corrutinas

En esta carpeta aprendes a escribir código **asíncrono** que se lee como si fuera secuencial: cómo esperar a la red o a la base de datos sin bloquear hilos, en qué hilo corre cada cosa, cómo se cancelan y fallan las tareas en grupo, y cómo armar trabajos que se repiten en segundo plano.

-----

## Antes de empezar

Conviene que ya hayas leído [Funciones](../05-funciones/README.md) (sobre todo lambdas) y [Null safety y errores](../07-null-safety-y-errores/README.md), porque la cancelación funciona con excepciones. Las funciones de orden superior y las lambdas con receptor se ven a fondo en `09-funcional`.

Si en algún momento aparece una palabra que no conoces, búscala en el [Glosario](Glosario.md).

-----

## Orden de lectura

Las lecciones se apoyan una en la otra, así que conviene leerlas en este orden:

| Lección | Qué aprendes | Qué necesitas saber antes |
| --- | --- | --- |
| [1. `suspend`](01-suspend.md) | Por qué existen las corrutinas, qué significa suspender, `runBlocking`, `launch` y `delay` | Funciones y lambdas |
| [2. Scopes y dispatchers](02-scopes-y-dispatchers.md) | `CoroutineScope`, `launch` vs `async`, `coroutineScope { }`, `Dispatchers` y `withContext` | `suspend` |
| [3. Concurrencia estructurada](03-concurrencia-estructurada.md) | Padres e hijos, cancelación cooperativa, `SupervisorJob`, `withTimeout` y manejo de errores | Scopes y excepciones |
| [4. Jobs programados](04-jobs-programados.md) | Tareas periódicas en un servidor Ktor: ciclo de vida, solapamiento y varias instancias | Concurrencia estructurada |

-----

## El mapa completo en una mirada

```
suspend                  ->  "esperar sin bloquear el hilo"
   suspend fun buscarPedido(id: String): Pedido
   delay(1.seconds)         // pausa la corrutina, no el hilo

Scopes y dispatchers     ->  "dónde vive y en qué hilo corre"
   launch { ... }           // dispara y olvida -> Job
   async { ... }.await()    // dispara y espera un valor -> Deferred
   withContext(Dispatchers.IO) { jdbc() }   // código bloqueante, al pool de IO

Concurrencia estructurada ->  "ningún hijo queda huérfano"
   coroutineScope { }       // si un hijo falla, cancela a los hermanos
   supervisorScope { }      // si un hijo falla, los demás siguen
   withTimeout(5.seconds) { ... }
   catch (e: CancellationException) -> SIEMPRE relanzarla

Jobs programados         ->  "repetir algo cada X tiempo"
   launch { while (isActive) { trabajo(); delay(intervalo) } }
   atado al ciclo de vida de la app, sin solaparse
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

* [Guía oficial de corrutinas](https://kotlinlang.org/docs/coroutines-guide.html): la referencia de `kotlinx.coroutines`.

-----

## Después de esta carpeta

Ya sabes ejecutar trabajo concurrente sin perder el control. Vuelve al índice: [README principal](../README.md).
