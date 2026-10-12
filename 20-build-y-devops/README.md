# Build y DevOps

En esta carpeta aprendes todo lo que rodea al código para que **llegue a funcionar en otro lado que no sea tu IntelliJ**: cómo Gradle compila un proyecto de varios módulos, cómo compartir su configuración sin copiarla, cómo levantar el entorno local completo con Docker, cómo trabajar con Git y merge requests, y cómo GitLab CI verifica cada cambio.

-----

## Antes de empezar

Conviene que ya hayas leído [DDD y monolito modular](../12-arquitectura/03-ddd-y-monolito-modular.md) (los módulos de Gradle), [Testing y calidad](../19-testing/README.md) (las tareas que corre el CI) y [Resiliencia e integraciones](../17-resiliencia-e-integraciones/README.md) (los servicios que levanta docker-compose).

Si en algún momento aparece una palabra que no conoces, búscala en el [Glosario](Glosario.md).

-----

## Orden de lectura

Las lecciones se apoyan una en la otra, así que conviene leerlas en este orden:

| Lección | Qué aprendes | Qué necesitas saber antes |
| --- | --- | --- |
| [1. Gradle con Kotlin DSL y version catalog](01-gradle-y-version-catalog.md) | Módulos, tareas, `implementation` vs `api`, wrapper, toolchains y `libs.versions.toml` | Monolito modular |
| [2. Convention plugins y composite builds](02-convention-plugins.md) | Compartir la configuración entre módulos con `build-logic` | Gradle |
| [3. Docker y docker-compose](03-docker-y-docker-compose.md) | Imágenes, contenedores, un Dockerfile para Ktor y el entorno local completo | Gradle |
| [4. Git y merge requests](04-git-y-merge-requests.md) | Commits atómicos, Conventional Commits, ramas, rebase y merge requests en GitLab | Nada |
| [5. GitLab CI](05-gitlab-ci.md) | `.gitlab-ci.yml`: etapas, caché, reportes, Testcontainers en CI e imágenes | Todas las anteriores |

-----

## El mapa completo en una mirada

```
Gradle          ->  "el que compila, testea y empaqueta"
   settings.gradle.kts: include(":pedidos", ":app")
   gradle/libs.versions.toml  ->  implementation(libs.ktor.server.core)

Convention      ->  "la configuración común, escrita una vez"
   build-logic/src/main/kotlin/tienda.kotlin-conventions.gradle.kts
   plugins { id("tienda.kotlin-conventions") }

Docker          ->  "el entorno, en un archivo"
   docker compose up -d   ->  postgres · redis · mailpit · minio (+ minio-init) · api · auth

Git             ->  "historia que se entiende"
   feat(pedidos): permitir cancelar pedidos pendientes
   rama corta -> merge request chico -> review -> merge

GitLab CI       ->  "cada cambio, verificado"
   lint -> test -> integración (Testcontainers) -> imagen -> deploy
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

* [Documentación de Gradle](https://docs.gradle.org/current/userguide/userguide.html), sobre todo [version catalogs](https://docs.gradle.org/current/userguide/version_catalogs.html) y [sharing build logic](https://docs.gradle.org/current/userguide/sharing_build_logic_between_subprojects.html)
* [Documentación de Docker Compose](https://docs.docker.com/compose/)
* [Conventional Commits](https://www.conventionalcommits.org/es/v1.0.0/)
* [Referencia de `.gitlab-ci.yml`](https://docs.gitlab.com/ci/yaml/)

-----

## Después de esta carpeta

Con esta etapa terminas el recorrido de backend. Lo que sigue en la wiki es Android, desde `21-...`. Vuelve al índice: [README principal](../README.md).
