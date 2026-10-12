# GitLab CI

## En una frase

GitLab CI ejecuta automáticamente, en cada push o merge request, los pasos que describes en `.gitlab-ci.yml` (formato, análisis, tests, tests de integración, imagen de Docker); si alguno falla, el cambio no se integra, y nadie depende de "me acordé de correr los tests".

-----

## Antes de empezar

Conviene que ya sepas:

* Las tareas de calidad y su orden: [Calidad: Kover, ktlint y detekt](../19-testing/05-calidad.md).
* Testcontainers: [Testcontainers](../19-testing/03-testcontainers.md).
* Gradle, Docker y merge requests: las lecciones anteriores de esta carpeta.

Palabras nuevas (también están en el [Glosario](Glosario.md)):

* **CI (integración continua):** Verificar automáticamente cada cambio integrándolo con el resto.
* **Pipeline:** El conjunto de jobs que corre para un commit.
* **Stage:** Una etapa del pipeline; los jobs de una etapa corren en paralelo, y las etapas en orden.
* **Job:** Un paso concreto del pipeline: un contenedor que ejecuta un script.
* **Runner:** La máquina que ejecuta los jobs.
* **Artifact:** Archivo que un job guarda al terminar (reportes, el JAR).
* **Caché:** Archivos que se reutilizan entre pipelines para no descargarlos de nuevo.
* **Docker-in-Docker (dind):** Un servicio de Docker que corre dentro del job, para que los tests puedan levantar contenedores.

-----

## Cómo funciona

1. Haces push a una rama o abres un merge request.
2. GitLab lee `.gitlab-ci.yml` y arma el **pipeline**.
3. Un **runner** ejecuta cada **job** dentro de un contenedor con la imagen que indiques.
4. Las **stages** corren en orden; los jobs de una misma stage, en paralelo.
5. Si un job falla, las stages siguientes no corren, y el MR muestra el pipeline en rojo.

## Un `.gitlab-ci.yml` completo

```yaml
stages:
  - verificar
  - test
  - empaquetar

default:
  image: eclipse-temurin:21-jdk

variables:
  GRADLE_USER_HOME: "$CI_PROJECT_DIR/.gradle"   # la caché de Gradle dentro del proyecto, para poder guardarla
  GRADLE_OPTS: "-Dorg.gradle.daemon=false"      # en CI, sin daemon

cache:
  key:
    files:
      - gradle/wrapper/gradle-wrapper.properties
      - gradle/libs.versions.toml
  paths:
    - .gradle/caches
    - .gradle/wrapper

workflow:
  rules:
    - if: $CI_PIPELINE_SOURCE == "merge_request_event"
    - if: $CI_COMMIT_BRANCH == $CI_DEFAULT_BRANCH

lint:
  stage: verificar
  script:
    - ./gradlew ktlintCheck detekt

test:
  stage: test
  script:
    - ./gradlew test
  artifacts:
    when: always
    reports:
      junit: "**/build/test-results/test/**/TEST-*.xml"

integracion:
  stage: test
  services:
    - docker:27-dind
  variables:
    DOCKER_HOST: "tcp://docker:2375"
    DOCKER_TLS_CERTDIR: ""
  script:
    - ./gradlew integrationTest koverVerify
  artifacts:
    when: always
    paths:
      - "**/build/reports/kover/"

imagen:
  stage: empaquetar
  image: docker:27
  services:
    - docker:27-dind
  rules:
    - if: $CI_COMMIT_BRANCH == $CI_DEFAULT_BRANCH
  script:
    - echo "$CI_REGISTRY_PASSWORD" | docker login -u "$CI_REGISTRY_USER" --password-stdin "$CI_REGISTRY"
    - docker build -t "$CI_REGISTRY_IMAGE/api:$CI_COMMIT_SHORT_SHA" .
    - docker push "$CI_REGISTRY_IMAGE/api:$CI_COMMIT_SHORT_SHA"
```

(Las versiones de las imágenes son de ejemplo; usa las que define tu equipo.)

## Las piezas, una por una

### Orden: de lo barato a lo caro

`lint` (segundos) corre primero. Si el formato está mal, no se gastan minutos en tests. Es el orden de [Calidad: Kover, ktlint y detekt](../19-testing/05-calidad.md).

`test` e `integracion` están en la misma stage: corren **en paralelo**.

### Caché de Gradle

Sin caché, cada pipeline descarga todas las dependencias de nuevo. La caché guarda `.gradle/caches` entre pipelines, y su **clave** depende del wrapper y del version catalog: si cambias una versión, se crea una caché nueva.

`GRADLE_USER_HOME` apunta dentro del proyecto porque GitLab solo puede guardar en caché rutas que estén dentro del directorio del proyecto.

### Reportes de tests

Con `artifacts: reports: junit`, GitLab lee los resultados de los tests y los muestra **en el MR**: cuántos pasaron, cuáles fallaron y su mensaje, sin abrir el log completo. `when: always` guarda el reporte incluso cuando el job falla, que es justo cuando más lo necesitas.

### Testcontainers en el CI

Los tests de integración necesitan levantar contenedores, pero el job **ya es** un contenedor. La solución común es el servicio **Docker-in-Docker**: un daemon de Docker que corre al lado del job, al que Testcontainers se conecta por `DOCKER_HOST`.

Depende de cómo esté configurado el runner (por ejemplo, dind necesita que el runner permita contenedores privilegiados). Si los tests de integración funcionan en tu máquina y fallan en el CI con errores de conexión a Docker, el problema casi siempre está acá, no en tu código.

### Reglas: cuándo corre cada cosa

* `workflow: rules` define cuándo se crea un pipeline: en cada MR y en cada commit a `main`.
* El job `imagen` tiene sus propias `rules`: solo corre en `main`. No tiene sentido publicar una imagen por cada rama.
* La imagen se etiqueta con el SHA corto del commit: cada imagen se puede rastrear hasta el código exacto que la generó.

### Variables y secretos

`$CI_REGISTRY`, `$CI_REGISTRY_USER`, `$CI_COMMIT_SHORT_SHA` y otras las define GitLab automáticamente.

Los secretos propios (claves de despliegue, tokens) se cargan en **Settings → CI/CD → Variables**, nunca en el YAML:

* **Masked:** GitLab los oculta si aparecen en el log.
* **Protected:** solo disponibles en ramas protegidas (como `main`), así un MR de cualquier rama no puede leerlos.

## Cuando el pipeline falla

1. **Abre el job que falló** y lee el log **desde el primer error**, no el último: el final suele ser una consecuencia.
2. **Mira el reporte de tests** en el MR: dice qué test falló y por qué.
3. **Reprodúcelo en tu máquina** con el mismo comando del `script` (`./gradlew ktlintCheck detekt`, `./gradlew test`).
4. Si pasa en tu máquina y falla en el CI, sospecha de lo que es **distinto**: versión de Java, zona horaria, Docker, orden de los tests, datos que tu máquina tiene y el CI no.

**Nunca** desactives un test para que el pipeline pase sin entender por qué falla. Un test que falla "a veces" (*flaky*) es un bug, del test o del código: muchas veces es una condición de carrera real.

## Más allá: despliegue

Después de `empaquetar` suele venir una stage `desplegar`, que toma la imagen publicada y la despliega en staging (automático) y en producción (con un botón manual, `when: manual`). Cómo se despliega depende de la plataforma de cada equipo, pero la idea es la misma: **lo que se despliega es exactamente la imagen que pasó todos los tests**.

-----

## Resumen en 5 líneas

1. `.gitlab-ci.yml` define stages en orden y jobs en paralelo; cada job corre en un contenedor de un runner.
2. De lo barato a lo caro: lint, tests, integración, imagen; con caché de Gradle dentro del proyecto.
3. Los reportes JUnit se ven en el MR; Testcontainers en el CI necesita un servicio Docker-in-Docker.
4. `rules` deciden cuándo corre cada job (la imagen, solo en `main`); los secretos van en variables masked y protected, nunca en el YAML.
5. Si falla, lee el primer error y reprodúcelo en local; nunca desactives un test que no entiendes.

-----

## Siguiente lección

Terminaste el recorrido de backend. Vuelve al índice: [README principal](../README.md).
