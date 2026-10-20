# Funciones de alcance: `let`, `run`, `with`, `apply` y `also`

## En una frase

Las funciones de alcance ejecutan un bloque de código **sobre un objeto**; las cinco hacen casi lo mismo y solo cambian en dos cosas: cómo se llama el objeto dentro del bloque (`this` o `it`) y qué devuelven (el objeto o el resultado del bloque).

-----

## Antes de empezar

Conviene que ya sepas:

* Lambdas con receptor: [Funciones de orden superior](01-funciones-de-orden-superior.md).
* Llamada segura `?.` y Elvis `?:`: [Nulabilidad](../07-null-safety-y-errores/01-nulabilidad.md).

Palabras nuevas (también están en el [Glosario](Glosario.md)):

* **Función de alcance:** Función de la librería estándar que ejecuta un bloque con un objeto como contexto.
* **Objeto de contexto:** El objeto sobre el que se ejecuta el bloque; se accede como `this` o como `it`.
* **Efecto secundario:** Algo que una función hace además de devolver un valor: imprimir, guardar, registrar en el log.

-----

## La tabla que lo resume todo

| Función | El objeto se llama | Devuelve | Uso típico |
| --- | --- | --- | --- |
| `let` | `it` | El resultado del bloque | Ejecutar algo solo si no es `null`; transformar |
| `run` | `this` | El resultado del bloque | Calcular un valor usando el objeto |
| `with(x)` | `this` | El resultado del bloque | Varias llamadas sobre un objeto que ya tienes |
| `apply` | `this` | **El objeto** | Configurar un objeto recién creado |
| `also` | `it` | **El objeto** | Hacer algo extra (log, validación) sin cambiar el flujo |

Una forma de recordarlo:

* **`apply` y `also` devuelven el objeto**: sirven para "hacerle algo y seguir con él".
* **`let`, `run` y `with` devuelven el resultado**: sirven para "calcular algo con él".

## `let`: con lo que no es `null`

El uso más común: ejecutar un bloque **solo si** el valor no es `null`.

```kotlin
val email: String? = buscarEmail(clienteId)

email?.let { enviarBienvenida(it) }  // si email es null, no se ejecuta nada
```

También sirve para transformar un nulable en otro tipo:

```kotlin
val id: PedidoId? = call.parameters["id"]?.let(::PedidoId)
```

## `apply`: configurar al crear

`apply` ejecuta el bloque con el objeto como `this` y **devuelve el mismo objeto**. Ideal para objetos que se configuran con propiedades:

```kotlin
val config = HikariConfig().apply {
  jdbcUrl = "jdbc:postgresql://localhost:5432/tienda"
  username = "tienda"
  maximumPoolSize = 10
}
```

Sin `apply`:

```kotlin
val config = HikariConfig()
config.jdbcUrl = "jdbc:postgresql://localhost:5432/tienda"
config.username = "tienda"
config.maximumPoolSize = 10
```

Lo mismo, pero `apply` agrupa la configuración en un bloque y te deja usar el resultado directamente. Es lo que en otros lenguajes resuelve el patrón **builder**, pero sin escribir una clase builder.

### La trampa: `apply` con objetos inmutables

`apply` devuelve **el objeto original**, no lo que calculaste adentro. Con objetos inmutables como `String`, eso tira el resultado a la basura:

```kotlin
val limpio = "  ana@mail.com  ".apply { trim() }
println("[$limpio]")   // [  ana@mail.com  ]  ← ¡los espacios siguen ahí!
```

`trim()` no modifica el `String` (ningún método de `String` lo hace): **devuelve uno nuevo**. Ese nuevo se pierde, y `apply` devuelve el original. Si quieres el resultado del bloque, la función correcta es `let` o `run`:

```kotlin
val limpio = "  ana@mail.com  ".let { it.trim() }   // o, más simple: "  ana@mail.com  ".trim()
```

`apply` tiene sentido cuando el bloque **cambia propiedades** del objeto (`jdbcUrl = ...`, `isRead = true`), no cuando llama funciones que devuelven un valor nuevo.

## `also`: hacer algo más sin cambiar nada

`also` recibe el objeto como `it` y lo devuelve tal cual. Sirve para meter un efecto secundario en medio de una cadena:

```kotlin
val pedido = crearPedido(command)
  .also { logger.info { "Pedido creado: ${it.id}" } }
```

Si borras el `also`, el código hace exactamente lo mismo, salvo el log. Esa es la señal de que `also` está bien usado.

## `run`: calcular con el objeto

`run` es como `let`, pero el objeto es `this`. Útil cuando el bloque llama varias funciones del objeto y devuelve un resultado:

```kotlin
val resumen = pedido.run {
  "Pedido $id de $clienteId por $total"   // id, clienteId y total son de this
}
```

Ojo con lo que asignas: con `run`, la variable recibe **la última expresión del bloque**, no el objeto.

```kotlin
data class Pedido(val id: String, val clienteId: String, val total: Double, var confirmado: Boolean = false)

val resultado = Pedido(id = "p1", clienteId = "c1", total = 500.0).run {
  confirmado = true
  "Pedido procesado: ${id.uppercase()}"
}
// resultado es un String, no un Pedido: el pedido se perdió
```

Si necesitas el objeto **y** un valor derivado, primero guarda el objeto en una variable y después llama a `run` sobre ella.

También existe `run` **sin** receptor, para agrupar un cálculo en una expresión:

```kotlin
val tasa = run {
  val base = 0.21
  val extra = if (esImportado) 0.05 else 0.0
  base + extra
}
```

## `with`: varias llamadas sobre algo que ya tienes

`with` no es una extensión: recibe el objeto como parámetro.

```kotlin
with(reporte) {
  agregarTitulo("Ventas")
  agregarTabla(pedidos)
  agregarPie("Generado el $fecha")
}
```

Como `run`, devuelve la última expresión, así que también sirve para calcular algo a partir del objeto:

```kotlin
val resumen = with(pedido) {
  confirmado = true
  "$id - $clienteId - confirmado: $confirmado"
}
```

¿`with` o `run`? Hacen lo mismo. `pedido.run { }` encaja al final de una cadena o con `?.run { }`; `with(pedido) { }` se lee como un bloque aparte, cómodo cuando son muchas operaciones sobre un objeto que ya tienes y que no es nulable.

## Combinarlas: un ejemplo corregido

Un ejercicio típico: validar un email que puede ser `null`, limpiándolo, registrándolo y devolviendo un `Boolean`. La versión que suele aparecer en los cursos es esta:

```kotlin
// ❌ tiene el bug de apply + trim
fun validarEmail(email: String?): Boolean =
  email?.let { texto ->
    texto.apply { trim() }                       // devuelve el texto SIN limpiar
      .also { println("Correo limpio: $it") }
      .run { contains("@") && contains(".") }
  } ?: false
```

Corregida, y sin funciones de alcance que no aportan:

```kotlin
// ✅
fun validarEmail(email: String?): Boolean {
  val limpio = email?.trim() ?: return false
  println("Correo limpio: $limpio")
  return "@" in limpio && "." in limpio
}
```

Y si quieres practicar la cadena, cada función en su rol:

```kotlin
fun validarEmail(email: String?): Boolean =
  email
    ?.let { it.trim() }                              // let: transforma (devuelve el String limpio)
    ?.also { println("Correo limpio: $it") }         // also: efecto secundario, devuelve lo mismo
    ?.run { contains("@") && contains(".") }         // run: calcula un Boolean usando this
    ?: false                                         // Elvis: si email era null, false
```

¿Por qué el `?: false` al final? Porque si `email` es `null`, toda la cadena con `?.` da `null`, y la función promete un `Boolean`. El operador Elvis pone el valor por defecto.

Fíjate que la versión con `if` y variables se lee igual de bien o mejor. Las funciones de alcance no son obligatorias: son una herramienta más.

## `takeIf` y `takeUnless`

Primas de las funciones de alcance: devuelven el objeto si cumple (o no cumple) una condición, y `null` si no.

```kotlin
val pagina = call.request.queryParameters["pagina"]
  ?.toIntOrNull()
  ?.takeIf { it > 0 }
  ?: 1
```

Se lee: "el número de página, si existe, es un número y es positivo; si no, 1".

## Cuándo **no** usarlas

Las funciones de alcance son azúcar sintáctica. Mal usadas, hacen el código **más** difícil de leer.

**Anidarlas:**

```kotlin
// ❌ ¿quién es `it` acá? ¿y `this`?
pedido?.let {
  it.cliente?.run {
    direccion?.also { println(it) }
  }
}
```

Si necesitas anidar, usa variables con nombre o `if`.

**`?.let` donde un `if` es más claro:**

```kotlin
// ❌ forzado
usuario?.let { u -> if (u.activo) notificar(u) }

// ✅ claro
if (usuario != null && usuario.activo) notificar(usuario)
```

**Elegir solo por estética:** si dudas, pregúntate qué necesitas **devolver**. Si es el objeto, `apply` o `also`. Si es otra cosa, `let` o `run`. Y si no devuelves nada y no hay nulables, probablemente no necesitas ninguna.

-----

## Resumen en 5 líneas

1. Las cinco funciones ejecutan un bloque sobre un objeto; cambian en cómo se llama el objeto (`this` o `it`) y en qué devuelven.
2. `apply` y `also` devuelven el objeto: `apply` para configurar propiedades (no sirve con `trim()` sobre un `String`), `also` para efectos secundarios como el log.
3. `let`, `run` y `with` devuelven el resultado del bloque, que puede ser de otro tipo: `?.let` es el idioma para "si no es null".
4. `takeIf` devuelve el objeto o `null` según una condición, y combina bien con `?:`.
5. No las anides ni las uses donde un `if` o una variable con nombre se leen mejor.

-----

## Siguiente lección

[3. `inline`](03-inline.md): qué cuesta una lambda y cómo lo evita Kotlin.
