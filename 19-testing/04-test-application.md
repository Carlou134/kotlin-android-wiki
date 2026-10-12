# `testApplication`: probar la API de Ktor

## En una frase

`testApplication` levanta tu aplicación Ktor **en memoria, sin red ni puerto**, y te da un cliente HTTP para hacerle peticiones reales; así pruebas rutas, serialización, autenticación y formato de errores tal como los vería un cliente.

-----

## Antes de empezar

Conviene que ya sepas:

* El módulo, los plugins y StatusPages: [Ktor server y plugins](../14-ktor-y-koin/01-ktor-server-y-plugins.md) y [Manejo de errores con StatusPages](../14-ktor-y-koin/02-manejo-de-errores-con-statuspages.md).
* Las rutas con autenticación: [Rutas y autenticación por ruta](../14-ktor-y-koin/03-rutas-y-autenticacion.md).
* Fakes y mocks: [Kotest](01-kotest.md) y [MockK](02-mockk.md).

Palabras nuevas (también están en el [Glosario](Glosario.md)):

* **`testApplication`:** Función de Ktor que levanta la aplicación en un motor de pruebas en memoria.
* **Test de contrato de API:** Test que verifica lo que ve el cliente: rutas, códigos de estado, JSON.
* **Par de claves de prueba:** Claves RSA generadas solo para los tests, para firmar tokens válidos.

-----

## Qué se prueba acá (y qué no)

| Se prueba con `testApplication` | Se prueba en otro lado |
| --- | --- |
| La ruta existe, con el método y el path correctos | Las reglas de negocio (tests de use cases) |
| El JSON de entrada y de salida | Las consultas SQL (Testcontainers) |
| Cada outcome se traduce al código HTTP correcto | |
| Sin token → `401`; token sin permiso → `403` | |
| El formato de error es el de toda la API | |

La ruta es un adapter: el test verifica la **traducción** entre HTTP y los puertos, no la lógica que hay detrás.

## Un primer test

```kotlin
class PedidoRoutesTest : FunSpec({

  test("POST /api/v1/pedidos crea un pedido y responde 201") {
    testApplication {
      application { moduloDePrueba(crearPedido = CrearPedidoFake()) }

      val cliente = createClient {
        install(ContentNegotiation) { json() }
      }

      val respuesta = cliente.post("/api/v1/pedidos") {
        contentType(ContentType.Application.Json)
        bearerAuth(tokenDePrueba(usuario = "u-1"))
        setBody(CrearPedidoRequest(lineas = listOf(LineaRequest("prod-1", 2))))
      }

      respuesta.status shouldBe HttpStatusCode.Created
      respuesta.body<PedidoResponse>().estado shouldBe "PENDIENTE"
    }
  }
})
```

* `application { ... }` configura la app como lo haría el servidor real.
* `createClient` crea un cliente que habla con la app en memoria, sin red; le instalas `ContentNegotiation` para enviar y recibir JSON.
* Las peticiones pasan por **todo** el pipeline: plugins, autenticación, StatusPages, rutas.

## Armar la app con piezas de prueba

Para no depender de la base ni de los proveedores, el módulo de prueba conecta las rutas reales con **use cases falsos**:

```kotlin
fun Application.moduloDePrueba(
  crearPedido: CrearPedido = mockk(),
  cancelarPedido: CancelarPedido = mockk(),
) {
  configurarSerializacion()
  configurarSeguridad(jwtConfigDePrueba)   // misma configuración, con la clave pública de prueba
  configurarErrores()
  routing {
    route("/api/v1") {
      authenticate("usuario") { pedidoRoutes(crearPedido, cancelarPedido) }
    }
  }
}
```

Esto funciona porque, como viste en [Koin](../14-ktor-y-koin/04-koin.md), las rutas **reciben** los puertos por parámetro y las funciones `configurarX()` reciben su configuración. Si las rutas pidieran sus dependencias a Koin por dentro, este test sería mucho más difícil.

Otra opción es instalar Koin en el test con módulos que reemplazan las definiciones reales por fakes.

## Tokens de prueba

La API verifica la firma del JWT. En los tests, generas un par de claves RSA **solo para los tests**, configuras el verificador con la clave pública y firmas los tokens con la privada:

```kotlin
object ClavesDePrueba {
  val rsa: RSAKey = RSAKeyGenerator(2048).keyID("test").generate()
}

fun tokenDePrueba(usuario: String, empresa: String = "e-1", expira: Instant = Clock.System.now() + 5.minutes): String {
  val claims = JWTClaimsSet.Builder()
    .issuer("https://auth.test")
    .audience("api.test")
    .subject(usuario)
    .claim("empresa", empresa)
    .expirationTime(Date.from(expira.toJavaInstant()))
    .build()
  return SignedJWT(JWSHeader.Builder(JWSAlgorithm.RS256).keyID("test").build(), claims)
    .apply { sign(RSASSASigner(ClavesDePrueba.rsa)) }
    .serialize()
}
```

Así pruebas también los casos malos:

```kotlin
test("sin token responde 401") {
  testApplication {
    application { moduloDePrueba() }
    client.get("/api/v1/pedidos").status shouldBe HttpStatusCode.Unauthorized
  }
}

test("con token vencido responde 401") {
  testApplication {
    application { moduloDePrueba() }
    client.get("/api/v1/pedidos") {
      bearerAuth(tokenDePrueba("u-1", expira = Clock.System.now() - 1.minutes))
    }.status shouldBe HttpStatusCode.Unauthorized
  }
}
```

## Cada outcome, su código

Con un mock que devuelve cada outcome, verificas la traducción a HTTP:

```kotlin
test("un pedido ya enviado responde 409 con el formato de error") {
  val cancelar = mockk<CancelarPedido>()
  coEvery { cancelar.ejecutar(any()) } returns CancelarPedidoOutcome.YaEnviado(Clock.System.now())

  testApplication {
    application { moduloDePrueba(cancelarPedido = cancelar) }
    val cliente = createClient { install(ContentNegotiation) { json() } }

    val respuesta = cliente.post("/api/v1/pedidos/p-1/cancelar") { bearerAuth(tokenDePrueba("u-1")) }

    respuesta.status shouldBe HttpStatusCode.Conflict
    respuesta.body<ErrorResponse>().codigo shouldBe "PEDIDO_YA_ENVIADO"
  }
}
```

## Cuidado con los tiempos

`testApplication` corre sobre la infraestructura de tests de corrutinas, que tiene un **límite de tiempo** por test (60 segundos en las versiones recientes de `kotlinx-coroutines-test`). Si dentro del test haces algo lento, como arrancar un contenedor de Testcontainers, puedes superar el límite y el test falla por timeout aunque el código esté bien.

* Arranca los contenedores **fuera** del bloque `testApplication` (por ejemplo, en un `object` compartido como en la [lección 3](03-testcontainers.md)).
* Mantén estos tests enfocados en HTTP, con fakes y mocks, para que sean rápidos.

## Test de contrato de las rutas

Con la app levantada en un test, también puedes recorrer **todas** las rutas registradas y verificar que cada una esté dentro de un bloque de autenticación o en una lista explícita de rutas públicas. Lo viste en [Rutas y autenticación por ruta](../14-ktor-y-koin/03-rutas-y-autenticacion.md): así ninguna ruta nueva queda pública por olvido.

-----

## Resumen en 5 líneas

1. `testApplication` levanta la app en memoria y su cliente recorre todo el pipeline: plugins, autenticación, StatusPages y rutas.
2. Testea la traducción HTTP ↔ puertos: rutas, JSON, códigos por outcome y formato de error; no la lógica de negocio.
3. Arma un módulo de prueba que conecte las rutas reales con use cases falsos; funciona porque las rutas reciben los puertos por parámetro.
4. Firma tokens con un par de claves RSA de prueba para probar `401` (sin token, vencido) y `403`.
5. Hay un límite de tiempo por test: arranca los contenedores afuera y mantén estos tests rápidos.

-----

## Siguiente lección

[5. Calidad: Kover, ktlint y detekt](05-calidad.md): las reglas que corren en cada build.
