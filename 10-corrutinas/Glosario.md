# Glosario

Las palabras que aparecen en las lecciones de esta carpeta, explicadas de forma simple. Están en orden alfabético.

-----

**Apagado ordenado (graceful shutdown).** Detener el servidor dejando que el trabajo en curso termine o se cancele de forma limpia.

**`async`.** Constructor de corrutinas que calcula un valor en paralelo y devuelve un `Deferred`.

**Bloquear.** Dejar un hilo parado esperando, sin poder hacer nada más.

**Cancelación cooperativa.** La corrutina no se mata de golpe; se le avisa y ella se detiene en el próximo punto de suspensión.

**`CancellationException`.** La excepción que se usa para cancelar una corrutina. No es un error: es la señal de "detente", y hay que relanzarla siempre.

**Concurrencia estructurada.** Organizar las corrutinas como un árbol de padres e hijas, donde ninguna queda sin dueño.

**Contexto de corrutina.** El conjunto de datos de una corrutina: su `Job`, su dispatcher, su nombre, etc.

**Corrutina.** Una tarea que puede pausarse y retomarse sin ocupar un hilo mientras espera.

**`CoroutineScope`.** El "dueño" de un grupo de corrutinas. Si se cancela el scope, se cancelan todas.

**`coroutineScope { }`.** Crea un scope dentro de una función `suspend`; espera a todas sus hijas y, si una falla, cancela a las demás.

**`Deferred`.** Un `Job` que además devuelve un valor, que se obtiene con `await()`.

**Dispatcher.** El componente que decide en qué hilo o grupo de hilos corre una corrutina (`Default`, `IO`, `Main`).

**`ensureActive()`.** Lanza `CancellationException` si la corrutina fue cancelada. Sirve en bucles sin puntos de suspensión.

**Fixed delay (intervalo fijo entre ejecuciones).** Esperar X tiempo *después* de que termina cada ejecución.

**Fixed rate (frecuencia fija).** Arrancar cada X tiempo, sin importar cuánto tardó la anterior.

**`GlobalScope`.** Scope global sin dueño ni cancelación. Es un antipatrón en código de aplicación.

**Hilo (thread).** Una línea de ejecución del sistema operativo. Son caros: cada uno ocupa memoria y hay pocos.

**Jitter.** Un pequeño retraso aleatorio que se suma para que muchas tareas no arranquen todas en el mismo instante.

**`Job`.** El identificador de una corrutina lanzada: sirve para esperarla (`join`) o cancelarla (`cancel`).

**Job programado.** Tarea que se ejecuta sola, cada cierto intervalo o a cierta hora.

**`launch`.** Constructor de corrutinas que dispara trabajo sin resultado y devuelve un `Job`.

**`NonCancellable`.** Contexto que permite ejecutar código `suspend` de limpieza aunque la corrutina ya esté cancelada.

**Punto de suspensión.** Una llamada a otra función `suspend`, donde la corrutina *puede* pausarse.

**`runBlocking`.** Constructor que bloquea el hilo actual hasta que terminan sus corrutinas. Solo para `main` y tests.

**`supervisorScope` / `SupervisorJob`.** Scope o `Job` cuyas hijas pueden fallar sin cancelar a sus hermanas.

**Suspender.** Pausar una corrutina en un punto concreto, liberando el hilo.

**Timeout.** Tiempo máximo que se le da a una operación antes de cancelarla (`withTimeout`, `withTimeoutOrNull`).

**`withContext`.** Ejecuta un bloque en otro dispatcher o contexto y devuelve su resultado.
