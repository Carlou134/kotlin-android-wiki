# `suspend`: esperar sin bloquear

## En una frase

Una función `suspend` puede **pausarse** mientras espera algo lento (la red, la base de datos) y liberar el hilo para que haga otro trabajo; cuando el resultado llega, retoma donde se quedó.

-----

## Antes de empezar

Conviene que ya sepas:

* Declarar funciones y lambdas: [Funciones](../05-funciones/01-funciones.md).

Palabras nuevas (también están en el [Glosario](Glosario.md)):

* **Hilo (thread):** Una línea de ejecución del sistema operativo. Son caros: cada uno ocupa memoria y hay pocos.
* **Bloquear:** Dejar un hilo parado esperando, sin poder hacer nada más.
* **Corrutina:** Una tarea que puede pausarse y retomarse sin ocupar un hilo mientras espera.
* **Suspender:** Pausar una corrutina en un punto concreto, liberando el hilo.
* **Punto de suspensión:** Una llamada a otra función `suspend`, donde la corrutina *puede* pausarse.

-----

## El problema: hilos esperando sin hacer nada

Un servidor recibe 1.000 peticiones a la vez. Cada una consulta una API externa que tarda 200 ms.

Con código bloqueante, cada petición ocupa **un hilo** durante esos 200 ms, aunque el hilo no haga nada más que esperar:

```kotlin
fun obtenerCotizacion(): Double {
  Thread.sleep(200) // simula la red: el hilo queda BLOQUEADO
  return 1050.0
}
```

1.000 peticiones = 1.000 hilos esperando. Los hilos son caros (alrededor de 1 MB de memoria cada uno), así que el servidor se queda sin recursos mucho antes de quedarse sin CPU.

## La solución: suspender en vez de bloquear

```kotlin
suspend fun obtenerCotizacion(): Double {
  delay(200) // la corrutina se PAUSA; el hilo queda libre
  return 1050.0
}
```

La diferencia:

| | `Thread.sleep(200)` | `delay(200)` |
| --- | --- | --- |
| ¿Qué se detiene? | El hilo entero | Solo la corrutina |
| ¿El hilo puede hacer otra cosa? | No | Sí, atiende otras corrutinas |
| ¿Cuántas en paralelo? | Pocos miles de hilos | Cientos de miles de corrutinas |

Por eso Ktor maneja cada petición HTTP en una corrutina: unos pocos hilos atienden miles de peticiones.

## La regla de `suspend`

Una función `suspend` **solo** se puede llamar desde otra función `suspend` o desde dentro de una corrutina:

```kotlin
fun main() {
  obtenerCotizacion() // ❌ no compila: main no es suspend
}
```

Para entrar al "mundo suspend" desde código normal necesitas un **constructor de corrutinas**. El más simple es `runBlocking`:

```kotlin
fun main() = runBlocking {
  val cotizacion = obtenerCotizacion() // ✅ estamos dentro de una corrutina
  println(cotizacion)
}
```

`runBlocking` bloquea el hilo actual hasta que termina todo lo de adentro. Sirve para `main()` y para tests, **nunca** dentro de código que ya es asíncrono (como una ruta de Ktor), porque bloquearía justo lo que querías liberar.

## El código se lee secuencial

Lo mejor de las corrutinas: el código asíncrono se escribe de arriba abajo, sin callbacks.

```kotlin
suspend fun procesarPedido(id: String) {
  val pedido = buscarPedido(id)         // espera a la base
  val cotizacion = obtenerCotizacion()  // espera a la API
  val total = pedido.total * cotizacion
  guardarTotal(id, total)               // espera a la base
}
```

Cada línea con una función `suspend` es un **punto de suspensión**: la corrutina puede pausarse ahí y retomar en la línea siguiente cuando llega el resultado.

## Lanzar varias corrutinas: `launch`

`launch` arranca una corrutina nueva que corre **a la vez** que las demás:

```kotlin
fun main() = runBlocking {
  launch {
    delay(1000)
    println("Terminó la tarea lenta")
  }
  launch {
    delay(500)
    println("Terminó la tarea rápida")
  }
  println("Lancé las dos")
}
```

Salida:

```
Lancé las dos
Terminó la tarea rápida
Terminó la tarea lenta
```

El programa tarda alrededor de 1 segundo en total, no 1,5: las dos esperas ocurren al mismo tiempo, y todo en **un solo hilo**.

## Ideas equivocadas comunes

* **"`suspend` hace que la función corra en otro hilo."** No. `suspend` solo dice que la función *puede* pausarse. En qué hilo corre lo decide el dispatcher (lección 2).
* **"Si marco una función como `suspend`, ya no bloquea."** No. Si adentro llamas a `Thread.sleep` o a un driver JDBC bloqueante, el hilo se bloquea igual. `suspend` no convierte código bloqueante en no bloqueante.

-----

## Resumen en 5 líneas

1. Bloquear un hilo esperando la red o la base desperdicia un recurso caro y escaso.
2. Una función `suspend` puede pausarse en cada punto de suspensión y liberar el hilo mientras espera.
3. Solo se llama desde otra función `suspend` o desde una corrutina; `runBlocking` es la puerta de entrada en `main` y en tests.
4. `launch` arranca corrutinas que corren a la vez, incluso en un solo hilo.
5. `suspend` no cambia de hilo ni vuelve no bloqueante el código que ya bloquea.

-----

## Siguiente lección

[2. Scopes y dispatchers](02-scopes-y-dispatchers.md): dónde vive cada corrutina y en qué hilo corre.
