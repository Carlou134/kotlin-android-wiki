# Ktor y Koin

En esta carpeta aprendes a **armar el servidor HTTP**: cómo arranca una aplicación Ktor, qué plugins instalar y en qué orden, cómo convertir errores en respuestas consistentes, cómo organizar rutas que declaran su autenticación, y cómo Koin conecta todas las piezas.

-----

## Antes de empezar

Conviene que ya hayas leído [Arquitectura](../12-arquitectura/README.md) (las rutas son adapters primarios y viven en el composition root), [Corrutinas](../10-corrutinas/README.md) (cada petición es una corrutina) y [JWT con RS256](../16-seguridad/02-jwt-rs256.md).

Si en algún momento aparece una palabra que no conoces, búscala en el [Glosario](Glosario.md).

-----

## Orden de lectura

Las lecciones se apoyan una en la otra, así que conviene leerlas en este orden:

| Lección | Qué aprendes | Qué necesitas saber antes |
| --- | --- | --- |
| [1. Ktor server y plugins](01-ktor-server-y-plugins.md) | Arranque, configuración, pipeline, plugins esenciales y plugins propios | Corrutinas y composition root |
| [2. Manejo de errores con StatusPages](02-manejo-de-errores-con-statuspages.md) | Excepciones a respuestas HTTP, formato de error único y validación de entrada | Plugins y outcomes |
| [3. Rutas y autenticación por ruta](03-rutas-y-autenticacion.md) | Organizar rutas por contexto, versionado, parámetros y autenticación explícita | StatusPages y JWT |
| [4. Inyección de dependencias con Koin](04-koin.md) | `single` vs `factory`, qualifiers, módulos por contexto, Koin en Ktor y verificación | Composition root |

-----

## El mapa completo en una mirada

```
Ktor server      ->  "un pipeline de plugins alrededor de tus rutas"
   fun Application.module() { configurarPlugins(); configurarSeguridad(); configurarRutas() }
   ContentNegotiation  CallId  CallLogging  CORS  Authentication  StatusPages

StatusPages      ->  "toda excepción termina en una respuesta prolija"
   exception<NoEncontradoException> { call, e -> call.respond(404, ErrorResponse(...)) }

Rutas            ->  "cada ruta dice cómo se autentica"
   route("/api/v1") {
     authenticate("usuario") { pedidoRoutes() }
     publico { healthRoutes() }
   }

Koin             ->  "un módulo por contexto, el root los junta"
   val pedidosModule = module { singleOf(::ExposedPedidoRepository) bind PedidoRepository::class }
   install(Koin) { modules(infraModule, pedidosModule) }
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

* [Documentación de Ktor server](https://ktor.io/docs/server-create-a-new-project.html)
* [Documentación de Koin para Ktor](https://insert-koin.io/docs/reference/koin-ktor/ktor)
* [ktor-hexagonal-multimodule](https://github.com/mobiletoly/ktor-hexagonal-multimodule): un módulo de Koin por módulo de Gradle (usa Ktor 2).

-----

## Después de esta carpeta

Ya sabes armar un servidor completo. Vuelve al índice: [README principal](../README.md).
