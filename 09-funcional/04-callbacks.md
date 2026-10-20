# Callbacks

## En una frase

Un callback es una función que le pasas a otra para que **ella decida cuándo llamarla**: típicamente una para "salió bien" y otra para "salió mal", así la función que trabaja no tiene que saber qué hacer con el resultado.

-----

## Antes de empezar

Conviene que ya sepas:

* Tipos de función, lambdas e `it`: [Funciones de orden superior](01-funciones-de-orden-superior.md).
* `find` y la diferencia con `first`: [Funciones de orden superior](01-funciones-de-orden-superior.md).
* Argumentos con nombre: [Funciones](../05-funciones/01-funciones.md).

Palabras nuevas (también están en el [Glosario](Glosario.md)):

* **Callback:** Función que se pasa como argumento para que otra la ejecute cuando ocurra algo (un éxito, un error, un resultado).
* **`Unit`:** El tipo de "no devuelve nada útil"; `() -> Unit` es una función que no recibe ni devuelve nada.
* **Callback hell:** Callbacks dentro de callbacks dentro de callbacks, hasta que el código se vuelve ilegible.

-----

## El problema: una función con dos finales

Quieres enviar un email. Puede salir bien o mal, y **quien llama** es el que sabe qué hacer en cada caso: una pantalla muestra un mensaje, un job lo reintenta, un test lo verifica.

Si la función decide sola, queda atada a un solo uso:

```kotlin
fun enviarEmail(email: Email) {
  if (email.remitente.contains("@")) {
    println("Enviado")        // ¿y si quien llama quería guardarlo en una base de datos?
  } else {
    println("Error")          // ¿y si quería reintentar?
  }
}
```

## La solución: pasar el "qué hacer" como parámetro

```kotlin
data class Email(val asunto: String, val remitente: String, val leido: Boolean = false)

fun enviarEmail(
  email: Email,
  onSuccess: (Email, String) -> Unit,
  onError: (String, Int) -> Unit,
) {
  if (email.remitente.contains("@")) {
    onSuccess(email, "Mensaje enviado a ${email.remitente}")
  } else {
    onError("Email inválido: ${email.remitente}", 400)
  }
}
```

Lee los tipos con calma:

* `onSuccess: (Email, String) -> Unit`: recibe el email y un mensaje; no devuelve nada.
* `onError: (String, Int) -> Unit`: recibe un mensaje y un código (400 es "bad request" en HTTP); no devuelve nada.

`enviarEmail` hace su trabajo y **avisa**. No sabe ni le importa qué pasa después.

## Llamarla

```kotlin
val email = Email(asunto = "Reunión", remitente = "jefe@empresa.com")

enviarEmail(
  email,
  onSuccess = { e, mensaje -> println(mensaje) },
  onError = { mensaje, codigo -> println("$mensaje, código $codigo") },
)
// Mensaje enviado a jefe@empresa.com
```

Con `remitente = "jefe.empresa"`, la salida cambia a `Email inválido: jefe.empresa, código 400`. Mismo llamado, otro camino.

Tres detalles de sintaxis:

* **Los callbacks tienen dos parámetros, así que no hay `it`:** hay que nombrarlos (`e, mensaje ->`).
* **Si no usas un parámetro, ponle `_`:** `onSuccess = { _, mensaje -> println(mensaje) }`. Le dice al lector "esto lo ignoro a propósito".
* **Argumentos con nombre:** con dos lambdas seguidas, `onSuccess =` y `onError =` evitan confundirlas. Solo la **última** podría ir como trailing lambda, y con dos callbacks eso suele leerse peor.

## Otro ejemplo: buscar con condición

Esta función combina todo: recibe la condición como lambda y dos callbacks para avisar el resultado.

```kotlin
fun buscarEmail(
  emails: List<Email>,
  condicion: (Email) -> Boolean,
  onFound: (Email) -> Unit,
  onNotFound: () -> Unit,
) {
  val encontrado = emails.find(condicion)
  if (encontrado != null) onFound(encontrado) else onNotFound()
}
```

* `condicion` se pasa directo a `find`: es del mismo tipo que lo que `find` espera.
* `find` devuelve `null` si no hay coincidencia (con `first`, explotaría).
* `onNotFound: () -> Unit` no recibe nada: solo avisa "no estaba".

```kotlin
buscarEmail(
  emails,
  condicion = { it.asunto.contains("urgente", ignoreCase = true) },
  onFound = { println("Encontrado: ${it.asunto}") },
  onNotFound = { println("No hay emails urgentes") },
)
```

Acá sí aparece `it`: cada una de esas lambdas tiene **un** parámetro (o ninguno, en `onNotFound`).

## Por qué esto importa: separar decisión de reacción

La función que trabaja (buscar, enviar, validar) **no decide qué hacer con el resultado**; solo lo notifica. Quien llama define la reacción.

Es la misma idea que los eventos en JavaScript (`boton.addEventListener("click", () => ...)`) y la base de muchas APIs de Android: `setOnClickListener { ... }` es un callback.

## El límite: callback hell

Los callbacks se complican cuando un paso depende del anterior:

```kotlin
// ❌ cada paso anida al siguiente
buscarCliente(id,
  onFound = { cliente ->
    crearPedido(cliente,
      onSuccess = { pedido ->
        enviarEmail(pedido.email,
          onSuccess = { _, _ -> println("Listo") },
          onError = { mensaje, _ -> println(mensaje) },
        )
      },
      onError = { println(it) },
    )
  },
  onNotFound = { println("No existe") },
)
```

Tres pasos y ya cuesta seguirlo. Kotlin tiene dos herramientas mejores para este caso:

| Alternativa | Qué cambia | Dónde se ve |
| --- | --- | --- |
| Devolver un resultado tipado (`sealed`) | La función **devuelve** "éxito o error" y quien llama usa `when` | [Sealed](../08-poo-avanzada/02-sealed.md) |
| Corrutinas (`suspend`) | Escribes los pasos uno debajo del otro, como código normal, aunque esperen a la red | [`suspend`](../10-corrutinas/01-suspend.md) |

```kotlin
// ✅ el mismo flujo con suspend: sin anidar
suspend fun procesar(id: String) {
  val cliente = buscarCliente(id) ?: return println("No existe")
  val pedido = crearPedido(cliente)
  enviarEmail(pedido.email)
  println("Listo")
}
```

¿Entonces los callbacks no sirven? Sí sirven: para **eventos** (un clic, un mensaje que llega) y para funciones simples con uno o dos finales. Para **secuencias de pasos**, las corrutinas ganan.

-----

## Resumen en 5 líneas

1. Un callback es una función que pasas como argumento para que otra la ejecute cuando ocurra algo.
2. El patrón `onSuccess` / `onError` separa al que trabaja del que decide qué hacer con el resultado.
3. Con dos o más parámetros no hay `it`: nómbralos, y usa `_` para los que ignoras.
4. Con varios callbacks, los argumentos con nombre (`onError = { }`) evitan confusiones.
5. Si los callbacks empiezan a anidarse, pasa a un resultado `sealed` o a corrutinas.

-----

## Siguiente lección

Terminaste la programación funcional. Lo próximo es [Corrutinas](../10-corrutinas/README.md), que resuelven el callback hell. Vuelve al índice: [README principal](../README.md).
