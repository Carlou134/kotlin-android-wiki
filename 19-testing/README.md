# Testing y calidad

En esta carpeta aprendes a **demostrar que tu código funciona y seguir demostrándolo** cada vez que alguien lo cambia: tests unitarios con Kotest, dobles de prueba con MockK, tests de integración contra un PostgreSQL real con Testcontainers, tests de la API completa con `testApplication`, y las herramientas que vigilan cobertura, formato y calidad en cada build.

-----

## Antes de empezar

Conviene que ya hayas leído [Arquitectura](../12-arquitectura/README.md), porque la arquitectura hexagonal es lo que vuelve el código fácil de testear, [Persistencia](../15-persistencia/README.md) y [Ktor y Koin](../14-ktor-y-koin/README.md).

Si en algún momento aparece una palabra que no conoces, búscala en el [Glosario](Glosario.md).

-----

## Orden de lectura

Las lecciones se apoyan una en la otra, así que conviene leerlas en este orden:

| Lección | Qué aprendes | Qué necesitas saber antes |
| --- | --- | --- |
| [1. Kotest](01-kotest.md) | La pirámide de tests, estilos de spec, matchers, tests con datos y testear use cases con fakes | Capa application |
| [2. MockK](02-mockk.md) | Mocks, stubs y fakes; `every`, `coEvery`, `verify`, `slot`; y cuándo **no** mockear | Kotest |
| [3. Testcontainers](03-testcontainers.md) | Tests de integración contra PostgreSQL real, migraciones, limpieza y concurrencia | Kotest, Exposed y Flyway |
| [4. `testApplication`](04-test-application.md) | Probar rutas, serialización, autenticación y errores de punta a punta en Ktor | Kotest, Ktor y JWT |
| [5. Calidad: Kover, ktlint y detekt](05-calidad.md) | Cobertura, formato, análisis estático y el orden en el pipeline | Todas las anteriores |

-----

## El mapa completo en una mirada

```
Kotest           ->  "tests que se leen como especificaciones"
   class CrearPedidoTest : FunSpec({ test("rechaza total cero") { ... shouldBe ... } })

MockK            ->  "dobles para lo que no quieres ejecutar de verdad"
   val notificador = mockk<Notificador>(relaxed = true)
   coVerify(exactly = 1) { notificador.enviar(any()) }

Testcontainers   ->  "la base de datos real, en un contenedor desechable"
   PostgreSQLContainer("postgres:16-alpine")  +  migraciones de Flyway

testApplication  ->  "la API completa, sin red"
   testApplication { application { module() }; client.post("/api/v1/pedidos") { ... } }

Calidad          ->  "reglas que corren en cada build"
   ktlintCheck  ->  detekt  ->  test  ->  koverVerify
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

* [Documentación de Kotest](https://kotest.io/docs/framework/framework.html)
* [Documentación de MockK](https://mockk.io/)
* [Testcontainers para Java](https://java.testcontainers.org/)
* [Testing en Ktor server](https://ktor.io/docs/server-testing.html)
* [Kover](https://kotlin.github.io/kotlinx-kover/gradle-plugin/), [ktlint](https://pinterest.github.io/ktlint/) y [detekt](https://detekt.dev/)

-----

## Después de esta carpeta

Ya sabes probar lo que construyes. Vuelve al índice: [README principal](../README.md).
