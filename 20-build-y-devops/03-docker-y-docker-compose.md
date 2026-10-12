# Docker y docker-compose

## En una frase

Docker empaqueta una aplicación con todo lo que necesita para correr en una **imagen** que funciona igual en cualquier máquina; docker-compose describe en un archivo **todos** los servicios del entorno (base de datos, Redis, correo, almacenamiento y tus apps) y los levanta con un comando.

-----

## Antes de empezar

Conviene que ya sepas:

* Las tareas de Gradle: [Gradle con Kotlin DSL y version catalog](01-gradle-y-version-catalog.md).
* Los servicios que usa el backend: [Redis](../17-resiliencia-e-integraciones/04-redis.md), [Almacenamiento S3 y MinIO](../17-resiliencia-e-integraciones/05-almacenamiento-s3-y-minio.md) y [Correo SMTP](../17-resiliencia-e-integraciones/06-correo-smtp.md).
* La configuración por variables de entorno: [Ktor server y plugins](../14-ktor-y-koin/01-ktor-server-y-plugins.md).

Palabras nuevas (también están en el [Glosario](Glosario.md)):

* **Imagen:** Plantilla de solo lectura con un sistema de archivos y un comando de arranque.
* **Contenedor:** Una instancia en ejecución de una imagen.
* **Dockerfile:** Archivo con los pasos para construir una imagen.
* **Build multi-etapa:** Dockerfile con varias etapas: una para compilar y otra, mínima, para ejecutar.
* **Volumen:** Almacenamiento que sobrevive cuando el contenedor se borra.
* **Fat JAR:** Un único archivo `.jar` con la app y todas sus dependencias.
* **Healthcheck:** Comando que Docker ejecuta para saber si un servicio está listo.

-----

## El problema: "instalá Postgres, Redis, MinIO..."

Sin Docker, la guía para empezar en el proyecto dice: instalá PostgreSQL 16 (no 15), creá dos bases, instalá Redis, configurá un servidor de correo, instalá MinIO, creá los buckets... Cada uno termina con versiones distintas, y "en mi máquina funciona" se vuelve una frase diaria.

Con docker-compose, la guía es: `docker compose up -d`.

## Imágenes y contenedores

* Una **imagen** es como una clase: una plantilla (por ejemplo, `postgres:16-alpine`).
* Un **contenedor** es como una instancia: la imagen corriendo, con su propio sistema de archivos y su red.
* Al borrar un contenedor, se pierde lo que escribió adentro, salvo que esté en un **volumen**.

## Un Dockerfile para una app Ktor

Primero se genera un fat JAR. El plugin de Gradle de Ktor tiene la tarea `buildFatJar`, que deja el archivo en `build/libs/` (con el sufijo `-all.jar`).

```dockerfile
# Etapa 1: compilar
FROM eclipse-temurin:21-jdk AS build
WORKDIR /src
COPY . .
RUN ./gradlew :app:buildFatJar --no-daemon

# Etapa 2: ejecutar (solo el JRE y el JAR)
FROM eclipse-temurin:21-jre
RUN useradd --system --create-home app
USER app
WORKDIR /app
COPY --from=build /src/app/build/libs/*-all.jar app.jar
EXPOSE 8080
ENTRYPOINT ["java", "-jar", "app.jar"]
```

* **Multi-etapa:** la imagen final tiene solo el JRE y el JAR, sin el JDK, el código fuente ni la caché de Gradle. Más chica y con menos superficie de ataque.
* **Usuario sin privilegios:** si alguien compromete la app, no es `root` dentro del contenedor.
* **Sin secretos en la imagen:** la configuración entra por variables de entorno al arrancar, como viste en [Ktor server y plugins](../14-ktor-y-koin/01-ktor-server-y-plugins.md). La misma imagen sirve para staging y producción.

Un `.dockerignore` (con `build/`, `.gradle/`, `.idea/`, `.git/`) evita copiar basura al contexto del build.

## El entorno local con docker-compose

`docker-compose.yml` en la raíz del repositorio:

```yaml
services:
  postgres:
    image: postgres:16-alpine
    environment:
      POSTGRES_DB: tienda
      POSTGRES_USER: tienda
      POSTGRES_PASSWORD: tienda
    ports: ["5432:5432"]
    volumes: [postgres-data:/var/lib/postgresql/data]
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U tienda"]
      interval: 5s
      retries: 10

  redis:
    image: redis:7-alpine
    ports: ["6379:6379"]

  mailpit:
    image: axllent/mailpit
    ports:
      - "1025:1025"   # SMTP
      - "8025:8025"   # interfaz web

  minio:
    image: minio/minio
    command: server /data --console-address ":9001"
    environment:
      MINIO_ROOT_USER: minio
      MINIO_ROOT_PASSWORD: minio12345
    ports:
      - "9000:9000"   # API S3
      - "9001:9001"   # consola web
    volumes: [minio-data:/data]

  minio-init:
    image: minio/mc
    depends_on: [minio]
    entrypoint: >
      /bin/sh -c "
      until mc alias set local http://minio:9000 minio minio12345; do sleep 1; done;
      mc mb --ignore-existing local/archivos;
      "

  api:
    build: .
    environment:
      DB_URL: jdbc:postgresql://postgres:5432/tienda
      REDIS_URL: redis://redis:6379
      SMTP_HOST: mailpit
      SMTP_PORT: "1025"
      S3_ENDPOINT: http://minio:9000
    ports: ["8100:8080"]
    depends_on:
      postgres: { condition: service_healthy }
      minio-init: { condition: service_completed_successfully }
    profiles: [apps]

  auth:
    build:
      context: .
      dockerfile: auth-app/Dockerfile
    ports: ["8101:8080"]
    depends_on:
      postgres: { condition: service_healthy }
    profiles: [apps]

volumes:
  postgres-data:
  minio-data:
```

Lo que conviene entender:

* **Los servicios se ven por su nombre.** Dentro de la red de compose, la API se conecta a `postgres:5432`, no a `localhost`. Desde tu máquina, sí es `localhost:5432` (por el `ports`).
* **`ports: "8100:8080"`** = puerto de tu máquina : puerto del contenedor.
* **`depends_on` con `condition`:** la API espera a que Postgres pase su healthcheck y a que `minio-init` termine. Sin eso, la app arranca antes que la base y falla.
* **`minio-init`** es un contenedor de un solo uso: espera a MinIO, crea el bucket y termina.
* **`profiles: [apps]`:** las apps solo se levantan si lo pides. El flujo diario suele ser levantar **solo la infraestructura** y correr la app desde IntelliJ, con el debugger.
* Las contraseñas de este archivo son **solo para desarrollo local**. Nunca reutilices estos valores en otro entorno.

## Comandos del día a día

```
docker compose up -d                  # levanta la infraestructura en segundo plano
docker compose --profile apps up -d   # también las apps
docker compose ps                     # qué está corriendo y su estado
docker compose logs -f api            # los logs de un servicio, en vivo
docker compose restart api            # reiniciar uno
docker compose down                   # apaga y borra los contenedores (los datos quedan en los volúmenes)
docker compose down -v                # ⚠️ también borra los volúmenes: base y archivos vacíos
docker compose up -d --build api      # reconstruir la imagen después de cambiar código
```

`down -v` es útil para empezar de cero (por ejemplo, para volver a correr todas las migraciones), pero borra **todos** los datos locales.

-----

## Resumen en 5 líneas

1. Una imagen es una plantilla; un contenedor es una imagen corriendo; lo que no esté en un volumen se pierde al borrarlo.
2. El Dockerfile de una app Ktor es multi-etapa: compila el fat JAR con el JDK y ejecuta solo con el JRE, sin `root` ni secretos.
3. docker-compose describe todo el entorno: Postgres, Redis, Mailpit, MinIO con un `minio-init` que crea los buckets, y las apps.
4. Dentro de compose los servicios se ven por nombre; `depends_on` con `condition` ordena el arranque usando healthchecks.
5. Flujo diario: `up -d` para la infraestructura y la app en IntelliJ; `down -v` borra todos los datos.

-----

## Siguiente lección

[4. Git y merge requests](04-git-y-merge-requests.md): cómo se trabaja en equipo sobre el mismo código.
