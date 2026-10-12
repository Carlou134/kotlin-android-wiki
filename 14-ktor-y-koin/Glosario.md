# Glosario

Las palabras que aparecen en las lecciones de esta carpeta, explicadas de forma simple. Están en orden alfabético.

-----

**`ApplicationCall`.** El objeto `call`: la petición y su respuesta en curso.

**`bind`.** En Koin, registra una definición también bajo el tipo de una interfaz.

**CallId.** Plugin de Ktor que asigna un identificador único a cada petición y lo devuelve en un header.

**CallLogging.** Plugin de Ktor que registra una línea de log por cada petición.

**ContentNegotiation.** Plugin de Ktor que convierte entre JSON (u otros formatos) y objetos de Kotlin.

**CORS.** Mecanismo del navegador que bloquea peticiones entre dominios a menos que el servidor las permita.

**Definición (Koin).** La regla que le dice a Koin cómo crear un tipo, como `single { ... }`.

**Engine.** El servidor HTTP de bajo nivel que usa Ktor: Netty, CIO o Jetty.

**`EngineMain`.** Forma de arrancar Ktor leyendo la configuración desde `application.yaml` o `application.conf`.

**`embeddedServer`.** Forma de arrancar Ktor configurando todo en código.

**`factory`.** En Koin, una instancia nueva cada vez que se pide.

**Formato de error.** La forma única que tienen todas las respuestas de error de la API.

**Grafo de dependencias.** El conjunto de piezas y quién necesita a quién.

**Handler.** La lambda que se ejecuta cuando llega una petición a una ruta.

**Koin.** Contenedor de inyección de dependencias para Kotlin, sin generación de código.

**Módulo de Ktor.** Una función `Application.module()` que configura la aplicación.

**Parámetro de path.** Un valor dentro de la URL, como el `{id}` de `/pedidos/{id}`.

**Pipeline.** La cadena de fases por la que pasa cada petición y su respuesta.

**Plugin.** Componente que se instala en el pipeline y agrega comportamiento a todas las peticiones.

**Principal.** El objeto que representa a quien se autenticó en la petición.

**Problem Details.** Estándar (RFC 9457) para el formato de errores en APIs HTTP.

**Qualifier.** Un nombre que distingue dos definiciones de Koin del mismo tipo, como `named("auth")`.

**Query parameter.** Un valor después del `?` en la URL, como `?pagina=2`.

**RequestValidation.** Plugin de Ktor que valida el cuerpo de la petición antes de que llegue a la ruta.

**Resources (plugin).** Plugin de Ktor que define las rutas como clases, con parámetros tipados.

**Ruta.** La combinación de método HTTP y path que atiende un handler, como `POST /api/v1/pedidos`.

**`single`.** En Koin, una sola instancia para toda la aplicación.

**StatusPages.** Plugin de Ktor que mapea excepciones y códigos de estado a respuestas.

**`verify()`.** Función de Koin que comprueba en un test que todas las dependencias del grafo están declaradas.
