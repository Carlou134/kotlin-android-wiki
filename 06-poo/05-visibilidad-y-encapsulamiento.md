# Visibilidad y encapsulamiento

## En una frase

El encapsulamiento es **esconder lo de adentro de una clase y exponer solo lo necesario**; en Kotlin se hace con los modificadores de visibilidad (`public`, `private`, `protected`, `internal`), que deciden desde dónde se puede usar cada propiedad o función.

-----

## Antes de empezar

Conviene que ya sepas:

* Propiedades, constructor y funciones miembro: [Clases](01-clases.md).
* Getters, setters y `field`: [Getters y setters](02-getters-y-setters.md).

Palabras nuevas (también están en el [Glosario](Glosario.md)):

* **Encapsulamiento:** Proteger el estado interno de un objeto para que solo se pueda cambiar a través de sus funciones.
* **Modificador de visibilidad:** Palabra que decide desde dónde se puede usar algo: `public`, `private`, `protected` o `internal`.
* **`private set`:** Hace que una propiedad se pueda leer desde afuera pero solo modificar desde adentro de la clase.
* **Invariante:** Una regla que el objeto siempre debe cumplir, como "el saldo nunca es negativo".

-----

## El problema: cualquiera puede romper tu objeto

```kotlin
class CuentaBancaria(var saldo: Double)

val cuenta = CuentaBancaria(100.0)
cuenta.saldo = -5000.0   // compila, y nadie validó nada
```

La clase **no puede defender sus reglas**. Si cualquier parte del programa puede escribir `saldo`, la validación "no se puede depositar un monto negativo" tendrías que repetirla en cada lugar que toca la cuenta. Y alguno se te va a olvidar.

## La solución: esconder el estado, exponer operaciones

```kotlin
class CuentaBancaria(private var saldo: Double) {

  fun depositar(monto: Double): String =
    if (esMontoValido(monto)) {
      saldo += monto
      "Depósito exitoso, nuevo saldo: $saldo"
    } else {
      "Monto inválido"
    }

  private fun esMontoValido(monto: Double) = monto > 0
}

val cuenta = CuentaBancaria(100.0)
println(cuenta.depositar(50.0))   // Depósito exitoso, nuevo saldo: 150.0
println(cuenta.depositar(-10.0))  // Monto inválido
// cuenta.saldo = -5000.0         ❌ no compila: saldo es private
```

* `saldo` es `private`: solo la propia clase lo ve.
* `depositar` es pública: es **la única puerta** para cambiar el saldo, y siempre valida.
* `esMontoValido` es `private`: es un detalle interno. A nadie de afuera le importa cómo se valida.

Ahora la regla vive en **un solo lugar**, y es imposible saltársela. Eso es el encapsulamiento.

## Los cuatro modificadores

| Modificador | ¿Desde dónde se ve? | Cuándo usarlo |
| --- | --- | --- |
| `public` | Desde cualquier lugar. **Es el valor por defecto**: si no escribes nada, es `public` | La API de la clase: lo que otros deben usar |
| `private` | Solo dentro de la clase (o del archivo, si está declarado fuera de una clase) | Estado interno y funciones auxiliares |
| `protected` | Dentro de la clase **y de sus hijas** | Lo que una clase hija necesita, pero el resto no |
| `internal` | Desde cualquier lugar del mismo **módulo** (el mismo proyecto de Gradle) | Código compartido dentro de un módulo que no quieres exponer a otros módulos |

La regla práctica: **empieza con todo lo más cerrado posible** y abre solo lo que haga falta. Abrir algo después es fácil; cerrarlo cuando ya hay código que lo usa, no.

## Leer sí, modificar no: `private set`

A veces quieres que desde afuera se pueda **ver** el saldo, pero no **cambiarlo**. Para eso no hace falta esconder toda la propiedad, solo su setter:

```kotlin
class CuentaBancaria(saldoInicial: Double) {
  var saldo: Double = saldoInicial
    private set

  fun depositar(monto: Double) {
    require(monto > 0) { "El monto debe ser positivo" }
    saldo += monto
  }
}

val cuenta = CuentaBancaria(100.0)
println(cuenta.saldo)    // ✅ 100.0: se puede leer
// cuenta.saldo = 0.0    ❌ no compila: el setter es private
```

Es el mismo setter de la lección anterior, pero con visibilidad `private`. Es de los patrones más usados en Kotlin.

## Visibilidad y herencia

Una clase hija **no ve** lo que el padre declaró `private`. Si quieres que la hija lo use, va `protected`:

```kotlin
open class Email(val asunto: String, private val contrasena: String) {
  protected fun firmar() = "firmado por $asunto"
}

class EmailSeguro(asunto: String, contrasena: String) : Email(asunto, contrasena) {
  fun enviar() {
    println(firmar())         // ✅ protected: la hija lo ve
    // println(contrasena)    ❌ no compila: private es solo de Email
  }
}

// EmailSeguro("Hola", "1234").firmar()   ❌ no compila: protected no se ve desde afuera
```

El `open` es necesario porque en Kotlin las clases no se pueden heredar por defecto. La herencia se ve a fondo en [Herencia e interfaces](../08-poo-avanzada/01-herencia-e-interfaces.md).

## `private` no es seguridad

Esto es importante: `private` protege tu código **de errores de programación**, no de un atacante. Dos ejemplos:

* **Una data class muestra todo en `toString()`, aunque sea privado:**

```kotlin
data class Usuario(val email: String, private val contrasena: String)

println(Usuario("ana@mail.com", "1234"))   // Usuario(email=ana@mail.com, contrasena=1234)
```

Si eso termina en un log, la contraseña quedó escrita en texto plano.

* **Con reflexión** (una herramienta para inspeccionar objetos mientras el programa corre) se puede leer un campo `private`.

Además, una contraseña **no se guarda** en un objeto, ni siquiera `private`: se guarda su hash, y no con algoritmos viejos como SHA-1 (aunque algunos cursos todavía lo usan como ejemplo). Cómo se hace bien se ve en [Contraseñas con Argon2id](../16-seguridad/01-contrasenas-con-argon2id.md).

## Un detalle sobre el dinero

En los ejemplos se usa `Double` para el saldo porque es simple. En código real **no se usa `Double` para dinero**: `0.1 + 0.2` da `0.30000000000000004`. Se usa `BigDecimal` o un `Long` con los centavos. Lo vas a ver al guardar montos en la base de datos, en [Exposed DSL](../15-persistencia/04-exposed-dsl.md).

-----

## Resumen en 5 líneas

1. Encapsular es esconder el estado interno y exponer solo las operaciones que respetan las reglas del objeto.
2. Todo es `public` por defecto; `private` lo limita a la clase, `protected` suma las hijas, `internal` lo abre a todo el módulo.
3. `var x ... private set` deja leer desde afuera pero modificar solo desde adentro.
4. Una clase hija no ve lo `private` del padre; para eso existe `protected`.
5. `private` evita errores, no ataques: una data class lo muestra en `toString()` y una contraseña nunca se guarda en texto plano.

-----

## Siguiente lección

Terminaste esta carpeta. Sigue con la siguiente: [Null safety y errores](../07-null-safety-y-errores/README.md).
