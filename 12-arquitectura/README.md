# Arquitectura

En esta carpeta aprendes a **organizar un backend para que la lógica de negocio no dependa del framework**: qué va en cada capa, quién puede depender de quién, cómo se parte un sistema grande en módulos y cómo hacer que el compilador y los tests te impidan romper esas reglas.

-----

## Antes de empezar

Conviene que ya hayas leído [Programación orientada a objetos](../06-poo/README.md) y [Null safety y errores](../07-null-safety-y-errores/README.md). También usas interfaces, `sealed interface` y extension functions, que se ven en `08-poo-avanzada`.

Si en algún momento aparece una palabra que no conoces, búscala en el [Glosario](Glosario.md).

-----

## Orden de lectura

Las lecciones se apoyan una en la otra, así que conviene leerlas en este orden:

| Lección | Qué aprendes | Qué necesitas saber antes |
| --- | --- | --- |
| [1. SOLID e inversión de dependencias](01-solid-y-dip.md) | Los cinco principios SOLID y por qué la "D" es la base de todo lo demás | Clases e interfaces |
| [2. Arquitectura hexagonal](02-arquitectura-hexagonal.md) | Dominio, application e infraestructura; puertos de entrada y de salida; adapters primarios y secundarios | SOLID |
| [3. DDD y monolito modular](03-ddd-y-monolito-modular.md) | Bounded contexts, shared kernel, Anti-Corruption Layer y cómo partir el proyecto en módulos de Gradle | Arquitectura hexagonal |
| [4. Composition root](04-composition-root.md) | El único lugar donde se conectan las piezas, e inyección de dependencias con Koin | Arquitectura hexagonal |
| [5. La capa application](05-capa-application.md) | Use cases, commands y outcomes con `sealed interface` | Arquitectura hexagonal |
| [6. Tests de arquitectura](06-tests-de-arquitectura.md) | Verificar las reglas de capas automáticamente con Konsist | Todas las anteriores |

-----

## El mapa completo en una mirada

```
SOLID / DIP          ->  "depende de abstracciones, no de detalles"
   CrearPedidoService(repo: PedidoRepository)   // interfaz, no Exposed

Hexagonal            ->  "el negocio en el centro, la tecnología en el borde"
   domain  <-  application  <-  infrastructure
   puerto de entrada: CrearPedido        puerto de salida: PedidoRepository
   adapter primario:  rutas de Ktor      adapter secundario: repositorio Exposed

DDD                  ->  "cada parte del negocio con su propio idioma"
   :pedidos   :facturacion   :identidad   + :shared (shared kernel)
   ACL: traduce el modelo de un tercero al tuyo

Composition root     ->  "un único lugar que arma todo"
   :app -> install(Koin) { modules(pedidosModule, facturacionModule) }

Capa application     ->  "entra un command, sale un outcome"
   fun ejecutar(cmd: CrearPedidoCommand): CrearPedidoOutcome   // sealed

Tests de arquitectura ->  "las reglas se verifican, no se recuerdan"
   Konsist: domain.dependsOnNothing()
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

* [ktor-hexagonal-multimodule](https://github.com/mobiletoly/ktor-hexagonal-multimodule): plantilla pública con Ktor, Exposed, Koin y módulos `core` / `adapters` / `infra`. Usa Ktor 2, así que tómala como referencia de **estructura**, no para copiar código.

-----

## Después de esta carpeta

Ya sabes dónde va cada pieza. Lo próximo es ver por dónde entra el mundo exterior a tu sistema: [HTTP y APIs](../13-http-y-apis/README.md). Vuelve al índice: [README principal](../README.md).
