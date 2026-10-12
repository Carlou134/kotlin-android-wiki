# MockK

## En una frase

MockK es la librería de mocks para Kotlin: crea objetos falsos de una interfaz o clase, les dices qué responder y después verificas cómo se usaron; es útil para **efectos** (enviar un email, publicar un evento), y peligrosa cuando se usa para todo.

-----

## Antes de empezar

Conviene que ya sepas:

* Testear use cases con fakes: [Kotest](01-kotest.md).

Palabras nuevas (también están en el [Glosario](Glosario.md)):

* **Doble de prueba:** Cualquier objeto que reemplaza a uno real en un test.
* **Stub:** Doble que devuelve respuestas preparadas.
* **Mock:** Doble que, además, registra cómo se lo llamó, para verificarlo después.
* **Fake:** Implementación simplificada pero funcional (por ejemplo, en memoria).
* **Slot:** Contenedor que captura el argumento con el que se llamó a un mock.

-----

## Tres tipos de dobles

| Tipo | Qué hace | Ejemplo |
| --- | --- | --- |
| **Fake** | Funciona de verdad, pero simplificado | Repositorio en un `Map` |
| **Stub** | Devuelve lo que le digas | "El proveedor responde batería 87" |
| **Mock** | Devuelve lo que le digas y registra las llamadas | "Verificá que se envió un email" |

MockK sirve para stubs y mocks. Para repositorios, los fakes suelen ser mejores (más abajo, por qué).

## Lo básico

```kotlin
val tasas = mockk<ServicioDeTasas>()

every { tasas.tasaPara("AR") } returns 0.21     // stub: qué responde

tasas.tasaPara("AR") shouldBe 0.21

verify(exactly = 1) { tasas.tasaPara("AR") }    // mock: cómo se llamó
```

* `mockk<T>()` crea el doble. Si llamas a una función que no configuraste, **lanza una excepción**: así no pasa nada sin que lo decidas.
* `every { ... } returns ...` configura la respuesta.
* `verify { ... }` comprueba que hubo una llamada (y cuántas, con `exactly`).

## Funciones `suspend`: `coEvery` y `coVerify`

```kotlin
val proveedor = mockk<ProveedorPort>()

coEvery { proveedor.obtenerEstado(any()) } returns ResultadoProveedor.Ok(estadoDePrueba)

// ...

coVerify { proveedor.obtenerEstado("R-1") }
```

Para funciones `suspend`, siempre las variantes con `co`.

## Matchers de argumentos

```kotlin
every { repo.buscar(any()) } returns null                         // cualquier argumento
every { tasas.tasaPara(eq("AR")) } returns 0.21                   // exactamente "AR"
every { tasas.tasaPara(match { it.startsWith("U") }) } returns 0.1 // con condición
```

## Lanzar excepciones y respuestas en secuencia

```kotlin
coEvery { proveedor.obtenerEstado(any()) } throws IOException("timeout")

// Primera llamada falla, segunda funciona: ideal para probar reintentos
coEvery { proveedor.obtenerEstado(any()) } throws IOException("timeout") andThen ResultadoProveedor.Ok(estado)
```

## Capturar argumentos con `slot`

A veces no alcanza con saber que se llamó: quieres **inspeccionar** con qué se llamó.

```kotlin
val notificador = mockk<NotificadorEmail>()
val emailEnviado = slot<Email>()
coEvery { notificador.enviar(capture(emailEnviado)) } just Runs

service.ejecutar(command)

emailEnviado.captured.para shouldBe "ana@mail.com"
emailEnviado.captured.asunto shouldContain "Tu pedido"
```

`just Runs` es la forma de decir "no devuelve nada" (`Unit`).

## Mocks relajados

```kotlin
val logger = mockk<AuditoriaPort>(relaxed = true)
```

Un mock relajado responde valores por defecto (`Unit`, `0`, `""`, `null`) a todo lo no configurado. Útil para dependencias que no te importan en ese test. Úsalo con cuidado: un mock relajado nunca te avisa de una llamada inesperada.

## Verificar que algo NO pasó

```kotlin
coVerify(exactly = 0) { notificador.enviar(any()) }
```

Tan importante como verificar que pasó: "si el pedido no se pudo cancelar, no se envía email".

## Cuándo mockear y cuándo no

**Buen uso: efectos hacia afuera.** Un email, un evento, una llamada a otro sistema. Lo que te importa es **que se haga**, y no tienes otra forma de observarlo.

**Mal uso: repositorios.**

```kotlin
// ❌ Frágil
every { repo.buscar(id) } returns pedido
every { repo.guardar(any()) } just Runs
service.ejecutar(command)
verify { repo.guardar(match { it.estado == CANCELADO }) }
```

Este test dice **cómo** está implementado el use case (primero `buscar`, después `guardar`), no **qué** hace. Si mañana el use case usa `actualizarEstado` en vez de `guardar`, el test se rompe aunque el comportamiento sea idéntico.

Con un fake, el test verifica el **resultado**: "después de cancelar, el pedido está cancelado". Sobrevive a cualquier cambio interno.

**Mal uso: mockear lo que no es tuyo.**

* No mockees `HttpClient`: usa `MockEngine` ([Ktor Client](../17-resiliencia-e-integraciones/03-ktor-client-y-apis-de-terceros.md)).
* No mockees Exposed ni JDBC: usa un PostgreSQL real con Testcontainers ([lección 3](03-testcontainers.md)).

Si mockeas una librería ajena, tu test verifica que la usas como **crees** que funciona, no como funciona de verdad.

**Una señal de alarma:** si un test necesita 6 mocks, el problema no es el test: la clase hace demasiadas cosas.

-----

## Resumen en 5 líneas

1. Fakes funcionan de verdad (en memoria); stubs devuelven respuestas; mocks además registran las llamadas.
2. `every`/`coEvery` configuran respuestas; `verify`/`coVerify` comprueban llamadas, incluso que algo **no** pasó con `exactly = 0`.
3. `slot` captura argumentos para inspeccionarlos; `andThen` encadena respuestas para probar reintentos.
4. Mockea efectos hacia afuera (emails, eventos); para repositorios usa fakes, que verifican resultados y no implementación.
5. No mockees librerías ajenas: `MockEngine` para HTTP, Testcontainers para la base.

-----

## Siguiente lección

[3. Testcontainers](03-testcontainers.md): tests contra una base de datos real.
