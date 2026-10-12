# Scopes y dispatchers

## En una frase

Un **scope** define cuánto vive una corrutina y quién la cancela; un **dispatcher** define en qué hilos corre.

-----

## Antes de empezar

Conviene que ya sepas:

* Qué es `suspend` y qué hace `launch`: [`suspend`](01-suspend.md).

Palabras nuevas (también están en el [Glosario](Glosario.md)):

* **`CoroutineScope`:** El "dueño" de un grupo de corrutinas. Si se cancela el scope, se cancelan todas.
* **`Job`:** El identificador de una corrutina lanzada: sirve para esperarla (`join`) o cancelarla (`cancel`).
* **`Deferred`:** Un `Job` que además devuelve un valor, que se obtiene con `await()`.
* **Dispatcher:** El componente que decide en qué hilo o grupo de hilos corre una corrutina.
* **Contexto de corrutina:** El conjunto de datos de una corrutina: su `Job`, su dispatcher, su nombre, etc.

-----

## Scopes: toda corrutina tiene dueño

`launch` y `async` no son funciones sueltas: son extensiones de `CoroutineScope`. Siempre se lanzan **dentro de un scope**.

```kotlin
fun main() = runBlocking { // this: CoroutineScope
  launch { /* hija de runBlocking */ }
}
```

En un servidor Ktor ya tienes scopes listos:

* Cada **petición HTTP** corre en una corrutina; si el cliente corta la conexión, se cancela.
* La **`Application`** es un `CoroutineScope` que vive mientras el servidor está prendido.

## `launch` vs `async`

| | `launch` | `async` |
| --- | --- | --- |
| Devuelve | `Job` | `Deferred<T>` |
| Para qué | Disparar trabajo sin resultado | Calcular un valor en paralelo |
| Cómo se espera | `job.join()` | `deferred.await()` |

El caso de uso típico de `async`: hacer varias consultas independientes **en paralelo** y juntar los resultados.

```kotlin
suspend fun armarResumen(clienteId: String): Resumen = coroutineScope {
  val pedidos = async { buscarPedidos(clienteId) }     // empieza ya
  val facturas = async { buscarFacturas(clienteId) }   // empieza ya, en paralelo

  Resumen(pedidos.await(), facturas.await())           // espera las dos
}
```

Si cada consulta tarda 300 ms, el resumen tarda alrededor de 300 ms en total, no 600.

## `coroutineScope { }`: un scope dentro de una función

Dentro de una función `suspend` no tienes un scope a mano. `coroutineScope { }` crea uno que:

1. Te deja usar `launch` y `async` adentro.
2. **No termina** hasta que terminan todas sus hijas.
3. Si una hija falla, cancela a las demás y relanza el error.

Es la forma correcta de hacer paralelismo dentro de una función `suspend`.

## El antipatrón: `GlobalScope`

```kotlin
fun enviarEmailBienvenida(email: String) {
  GlobalScope.launch { enviarEmail(email) } // ❌
}
```

`GlobalScope` no tiene dueño: nadie la cancela, nadie espera a que termine y nadie se entera si falla. Si el servidor se apaga a mitad del envío, el email se pierde sin dejar rastro. Usa siempre un scope con ciclo de vida (lección 4).

## Dispatchers: en qué hilo corre

| Dispatcher | Hilos | Para qué |
| --- | --- | --- |
| `Dispatchers.Default` | Tantos como núcleos de CPU | Cálculo intensivo: ordenar, parsear, encriptar |
| `Dispatchers.IO` | Un pool más grande (64 por defecto) | Código **bloqueante**: JDBC, archivos, librerías viejas |
| `Dispatchers.Main` | El hilo de la UI | Solo en Android: actualizar la pantalla |
| `Dispatchers.Unconfined` | El que esté | Casos muy especiales; evítalo |

## `withContext`: cambiar de dispatcher

`withContext` ejecuta un bloque en otro dispatcher y vuelve con el resultado:

```kotlin
suspend fun buscarPedido(id: String): Pedido? =
  withContext(Dispatchers.IO) {
    // JDBC bloquea el hilo, así que lo mandamos al pool de IO
    jdbcBuscarPedido(id)
  }
```

¿Por qué importa en un backend? Los drivers **JDBC son bloqueantes**. Exposed sobre JDBC bloquea el hilo mientras espera a PostgreSQL. Si lo corres en los pocos hilos que atienden peticiones, los dejas tomados y el servidor deja de responder. En el pool de IO, el bloqueo no afecta al resto.

Exposed trae funciones para abrir transacciones desde código `suspend` (`newSuspendedTransaction` y, en versiones nuevas, `suspendTransaction`) que reciben el dispatcher como parámetro.

## Limitar la concurrencia

A veces no quieres que corran 64 cosas a la vez contra un recurso, por ejemplo una API externa que acepta 4 conexiones:

```kotlin
private val apiProveedor = Dispatchers.IO.limitedParallelism(4)

suspend fun consultarProveedor(id: String) = withContext(apiProveedor) {
  clienteBloqueante.get(id)
}
```

-----

## Resumen en 5 líneas

1. Toda corrutina vive en un `CoroutineScope`, que decide cuándo se cancela; en Ktor, la petición y la `Application` ya son scopes.
2. `launch` dispara trabajo y devuelve un `Job`; `async` calcula un valor y devuelve un `Deferred` que se espera con `await()`.
3. `coroutineScope { }` permite paralelismo dentro de una función `suspend` y espera a todas sus hijas.
4. `Default` es para CPU, `IO` es para código bloqueante como JDBC; `withContext` cambia de uno a otro.
5. Nunca uses `GlobalScope`: una corrutina sin dueño se pierde sin que nadie se entere.

-----

## Siguiente lección

[3. Concurrencia estructurada](03-concurrencia-estructurada.md): qué pasa cuando una corrutina falla o se cancela.
