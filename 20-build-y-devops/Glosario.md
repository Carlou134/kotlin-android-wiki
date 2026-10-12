# Glosario

Las palabras que aparecen en las lecciones de esta carpeta, explicadas de forma simple. Están en orden alfabético.

-----

**`api` (configuración).** Dependencia que se vuelve transitiva: la ven también los módulos que dependen del tuyo.

**Artifact.** Archivo que un job de CI guarda al terminar: reportes, el JAR.

**`build-logic`.** Carpeta con un build de Gradle independiente que contiene los convention plugins.

**Build multi-etapa.** Dockerfile con varias etapas: una para compilar y otra, mínima, para ejecutar.

**Caché (CI).** Archivos que se reutilizan entre pipelines para no descargarlos de nuevo.

**CI (integración continua).** Verificar automáticamente cada cambio integrándolo con el resto.

**Commit atómico.** Un commit que contiene un solo cambio lógico, completo y que compila.

**Composite build.** Un build de Gradle que incluye a otro build completo con `includeBuild`.

**Conflicto.** Cuando dos ramas cambiaron las mismas líneas y Git no puede decidir solo.

**Contenedor.** Una instancia en ejecución de una imagen de Docker.

**Conventional Commits.** Convención para los mensajes de commit: `tipo(alcance): descripción`.

**Convention plugin.** Plugin de Gradle propio que aplica una configuración común a varios módulos.

**Dependencia transitiva.** Una dependencia de tu dependencia.

**docker-compose.** Herramienta que describe y levanta varios contenedores relacionados desde un archivo YAML.

**Docker-in-Docker (dind).** Un servicio de Docker que corre dentro de un job de CI, para que los tests puedan levantar contenedores.

**Dockerfile.** Archivo con los pasos para construir una imagen.

**Fat JAR.** Un único archivo `.jar` con la app y todas sus dependencias.

**Gradle.** Herramienta de build: compila, corre tests, empaqueta y resuelve dependencias.

**Gradle wrapper.** Script (`gradlew`) que descarga y usa la versión exacta de Gradle del proyecto.

**Healthcheck.** Comando que Docker ejecuta para saber si un servicio está listo.

**Imagen.** Plantilla de solo lectura con un sistema de archivos y un comando de arranque.

**`implementation` (configuración).** Dependencia que usa tu módulo sin exponerla a los módulos que dependen de él.

**Job.** Un paso concreto de un pipeline de CI: un contenedor que ejecuta un script.

**Merge request (MR).** En GitLab, la propuesta de integrar una rama en otra, con revisión y CI.

**Módulo (subproyecto).** Una parte del proyecto de Gradle con su propio `build.gradle.kts`.

**Pipeline.** El conjunto de jobs de CI que corre para un commit.

**Precompiled script plugin.** Un archivo `.gradle.kts` dentro de `build-logic` que Gradle convierte en un plugin.

**Rama (branch).** Línea de trabajo paralela que sale de otra, normalmente `main`.

**Rebase.** Reaplicar tus commits encima de otra base, reescribiendo su historia.

**Runner.** La máquina que ejecuta los jobs de CI.

**Squash.** Juntar varios commits en uno solo, por ejemplo al mergear un MR.

**Stage.** Una etapa de un pipeline; sus jobs corren en paralelo y las stages, en orden.

**Tarea (task).** Una unidad de trabajo de Gradle: `compileKotlin`, `test`, `build`.

**Toolchain.** La versión de JDK con la que Gradle compila, independiente de la instalada.

**Version catalog.** Archivo central (`libs.versions.toml`) con las versiones y coordenadas de las dependencias.

**Volumen.** Almacenamiento de Docker que sobrevive cuando el contenedor se borra.
