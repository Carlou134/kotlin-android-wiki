# Kotest

## En una frase

Kotest es un framework de testing para Kotlin en el que los tests se escriben como **especificaciones legibles**, con matchers expresivos (`resultado shouldBe 5`) y soporte nativo para corrutinas.

-----

## Antes de empezar

Conviene que ya sepas:

* Use cases, commands y outcomes: [La capa application](../12-arquitectura/05-capa-application.md).
* Por qué los puertos permiten implementaciones en memoria: [SOLID e inversión de dependencias](../12-arquitectura/01-solid-y-dip.md).

Palabras nuevas (también están en el [Glosario](Glosario.md)):

* **Test unitario:** Prueba una pieza aislada (un use case, una función), sin base de datos ni red.
* **Test de integración:** Prueba una pieza junto con algo real: una base de datos, un servidor.
* **Spec:** Una clase de Kotest que agrupa tests.
* **Matcher:** Una función de aserción, como `shouldBe` o `shouldThrow`.
* **Fake:** Implementación real pero simplificada de un puerto, por ejemplo un repositorio en memoria.
* **Arrange, Act, Assert:** Estructura de un test: preparar, ejecutar y verificar.

-----

## La pirámide de tests

```
        /\        End-to-end: pocos, lentos, frágiles
       /  \
      /----\      Integración: base real, API completa
     /      \
    /--------\    Unitarios: muchos, rápidos (milisegundos)
```

* **Unitarios:** las reglas de negocio y los use cases. Son la base: miles en segundos.
* **Integración:** los adapters contra la tecnología real (repositorios contra PostgreSQL, rutas contra Ktor).
* **End-to-end:** el sistema completo desplegado. Pocos, para los flujos críticos.

La arquitectura hexagonal hace barata la base de la pirámide: como el núcleo solo depende de interfaces, se testea con fakes en memoria, sin levantar nada.

## Un primer test

```kotlin
class PedidoTest : FunSpec({

  test("un pedido no puede tener total cero") {
    shouldThrow<IllegalArgumentException> {
      Pedido(PedidoId("p1"), ClienteId("c1"), total = 0.0)
    }
  }

  test("el total con impuestos suma la tasa") {
    val pedido = Pedido(PedidoId("p1"), ClienteId("c1"), total = 100.0)

    pedido.totalConImpuestos(0.21) shouldBe 121.0
  }
})
```

* La clase extiende un **estilo de spec** (`FunSpec`) y los tests van en la lambda del constructor.
* Cada `test("...")` tiene un nombre en lenguaje natural que describe el **comportamiento**, no el método.
* Los cuerpos de los tests son `suspend`: puedes llamar funciones `suspend` directamente, sin `runBlocking`.

Kotest corre sobre la plataforma de JUnit, así que en Gradle se activa con `tasks.test { useJUnitPlatform() }`.

## Estilos de spec

Kotest tiene varios estilos; todos hacen lo mismo y cambia la forma de escribir:

```kotlin
// FunSpec: el más simple
class A : FunSpec({ test("hace algo") { } })

// StringSpec: el nombre es el test
class B : StringSpec({ "hace algo" { } })

// DescribeSpec: agrupa con describe/it
class C : DescribeSpec({
  describe("cancelar pedido") {
    it("rechaza pedidos ya enviados") { }
  }
})

// BehaviorSpec: Given / When / Then
class D : BehaviorSpec({
  given("un pedido enviado") {
    `when`("el cliente lo cancela") {
      then("se rechaza") { }
    }
  }
})
```

Elige **uno** para el proyecto y úsalo en todos lados. `FunSpec` es una buena opción por defecto.

## Matchers

```kotlin
resultado shouldBe 42
lista shouldContainExactly listOf("a", "b")
lista.shouldBeEmpty()
valor.shouldBeNull()
texto shouldStartWith "Hola"
outcome.shouldBeInstanceOf<CancelarPedidoOutcome.Cancelado>()   // y hace smart cast
shouldThrow<IllegalArgumentException> { ... }.message shouldBe "Total inválido"
```

Cuando un matcher falla, el mensaje dice qué esperaba y qué obtuvo, así no necesitas depurar para entender el error.

`shouldBeInstanceOf` devuelve el valor ya convertido al tipo, ideal para outcomes `sealed`:

```kotlin
val cancelado = outcome.shouldBeInstanceOf<CancelarPedidoOutcome.Cancelado>()
cancelado.pedido.estado shouldBe EstadoPedido.CANCELADO
```

## Testear un use case con fakes

Esta es la mayoría de tus tests. El repositorio real se reemplaza por uno en memoria:

```kotlin
class PedidoRepositoryEnMemoria : PedidoRepository {
  private val pedidos = mutableMapOf<PedidoId, Pedido>()

  override suspend fun buscar(id: PedidoId) = pedidos[id]
  override suspend fun guardar(pedido: Pedido) { pedidos[pedido.id] = pedido }
}
```

```kotlin
class CancelarPedidoServiceTest : FunSpec({

  val repo = PedidoRepositoryEnMemoria()
  val service = CancelarPedidoService(repo)

  test("cancela un pedido pendiente del mismo cliente") {
    // Arrange
    val pedido = unPedido(estado = EstadoPedido.PENDIENTE, cliente = ClienteId("c1"))
    repo.guardar(pedido)

    // Act
    val outcome = service.ejecutar(CancelarPedidoCommand(pedido.id, ClienteId("c1"), "ya no lo quiero"))

    // Assert
    outcome.shouldBeInstanceOf<CancelarPedidoOutcome.Cancelado>()
    repo.buscar(pedido.id)?.estado shouldBe EstadoPedido.CANCELADO
  }

  test("no deja cancelar el pedido de otro cliente") {
    val pedido = unPedido(cliente = ClienteId("c1"))
    repo.guardar(pedido)

    val outcome = service.ejecutar(CancelarPedidoCommand(pedido.id, ClienteId("otro"), "x"))

    outcome shouldBe CancelarPedidoOutcome.NoEsTuyo
  }
})
```

**Un test por outcome.** Si el use case tiene 4 outcomes, debería haber al menos 4 tests.

`unPedido(...)` es una **función de fábrica de test** con valores por defecto: cada test especifica solo lo que le importa y el resto se completa solo.

```kotlin
fun unPedido(
  id: PedidoId = PedidoId(UUID.randomUUID().toString()),
  cliente: ClienteId = ClienteId("c-default"),
  estado: EstadoPedido = EstadoPedido.PENDIENTE,
  total: Double = 100.0,
) = Pedido(id, cliente, total, estado)
```

## Aislamiento entre tests

Ojo con el ejemplo anterior: `repo` se crea **una vez** para todo el spec, así que lo que guarda un test lo ve el siguiente. Opciones:

* Limpiar en `beforeTest { repo.limpiar() }`.
* Crear el fake dentro de cada test.
* Configurar el spec con `isolationMode = IsolationMode.InstancePerTest`, que crea una instancia nueva del spec por test.

Un test que depende del orden en que corren los demás es un test roto que todavía no falló.

## Tests con datos

Cuando el mismo test aplica a muchos casos:

```kotlin
context("emails inválidos") {
  withData("", "ana", "ana@", "@mail.com") { texto ->
    shouldThrow<IllegalArgumentException> { Email(texto) }
  }
}
```

Cada valor aparece como un test separado en el reporte. (`withData` viene del módulo de data-driven testing de Kotest).

-----

## Resumen en 5 líneas

1. La pirámide: muchos unitarios rápidos, algunos de integración, pocos end-to-end.
2. Kotest escribe los tests como specs (`FunSpec`, `DescribeSpec`, `BehaviorSpec`); elige un estilo para todo el proyecto.
3. Matchers como `shouldBe`, `shouldThrow` y `shouldBeInstanceOf` (que además hace smart cast); los cuerpos son `suspend`.
4. Los use cases se testean con fakes en memoria de los puertos, un test por outcome y fábricas de datos con valores por defecto.
5. Cada test es independiente: limpia el estado o aísla las instancias; `withData` repite un test con muchos valores.

-----

## Siguiente lección

[2. MockK](02-mockk.md): dobles de prueba que verifican interacciones.
