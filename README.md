# 🎯 Kotlin & Android Native Development Wiki

Repositorio personal de apuntes, guías rápidas (*cheatsheets*) y patrones de arquitectura para el ecosistema de Kotlin, desarrollo móvil nativo en Android y la integración con servicios Backend.

---

## 🗺️ Ruta de aprendizaje

Cada carpeta es una etapa con su propio `README.md` (orden de lectura y mapa de conceptos) y `Glosario.md`. Conviene leerlas en orden:

| Etapa | Qué aprendes |
| --- | --- |
| [01. Fundamentos](01-fundamentos/README.md) | Qué es Kotlin, `main()`, compilador y JVM, entorno IntelliJ |
| [02. Tipos y variables](02-tipos-y-variables/README.md) | `val`/`var`, tipos, texto y números |
| [03. Colecciones](03-colecciones/README.md) | Listas, sets y maps |
| [04. Control de flujo](04-control-de-flujo/README.md) | Condicionales, `when`, rangos y bucles |
| [05. Funciones](05-funciones/README.md) | Funciones, argumentos, lambdas y recursión |
| [06. Programación orientada a objetos](06-poo/README.md) | Clases, getters/setters, data classes y enums |
| [07. Null safety y errores](07-null-safety-y-errores/README.md) | Tipos nulables, `?.`, `?:` y excepciones |
| [08. POO avanzada](08-poo-avanzada/README.md) | Herencia, interfaces, `sealed`, extension functions, `value class`, `object` |
| [09. Funcional](09-funcional/README.md) | Lambdas, funciones de alcance, `inline` |
| [10. Corrutinas](10-corrutinas/README.md) | `suspend`, scopes, dispatchers, concurrencia estructurada |
| [11. Serialización y tiempo](11-serializacion-y-tiempo/README.md) | kotlinx.serialization, kotlinx.datetime |
| [12. Arquitectura](12-arquitectura/README.md) | SOLID, hexagonal, DDD, composition root, use cases, Konsist |
| [13. HTTP y APIs](13-http-y-apis/README.md) | HTTP a fondo, REST, paginación, OpenAPI |
| [14. Ktor y Koin](14-ktor-y-koin/README.md) | Plugins, StatusPages, rutas con auth, DI |
| [15. Persistencia](15-persistencia/README.md) | SQL, transacciones, Postgres, Exposed, Flyway, bloqueos |
| [16. Seguridad](16-seguridad/README.md) | Argon2id, JWT, JWKS, refresh tokens, RBAC, AES-GCM, presigned URLs |
| [17. Resiliencia e integraciones](17-resiliencia-e-integraciones/README.md) | Backoff, idempotencia, Ktor Client, Redis, S3, SMTP |
| [18. Observabilidad](18-observabilidad/README.md) | Logging estructurado, correlation ID, métricas |
| [19. Testing](19-testing/README.md) | Kotest, MockK, Testcontainers, `testApplication`, calidad |
| 20. Build y DevOps *(pendiente)* | Gradle, convention plugins, Docker, Git, CI |

Las etapas de Android (Compose, Room, WorkManager) continúan desde `21-...`.

---

## 📂 Hoja de ruta

### 1. Kotlin Core & Asincronía
- **Fundamentos y Tipado:** Manejo estricto de nulos (*Null Safety*), `smart casts` y tipos de datos.
- **Programación Orientada a Objetos:** Interfaces, `sealed classes`, `data classes` y funciones de extensión.
- **Programación Funcional:** Lambdas, *inline functions* y funciones de alcance (`let`, `run`, `apply`, `also`, `with`).
- **Corrutinas y Programación Asíncrona:** *Dispatchers*, *Scopes*, cancelación cooperativa y reactividad con `Flow` y `StateFlow`.

### 2. Android Native Dev & Arquitectura
- **Arquitectura Limpia y MVVM:** Separación de capas (UI, Domain, Data), desacoplamiento con Inyección de Dependencias (**Hilt / Koin**).
- **UI Moderna con Jetpack Compose:** Declaración de interfaces, *Recomposition*, estados (`remember`, `mutableStateOf`) y tematización (*Material 3*).
- **Gestión de Estado y Ciclo de Vida:** `ViewModel`, manejo de eventos con `SharedFlow` y estados de pantalla UI con `StateFlow`.
- **Consumo de APIs & Persistencia Local:** Peticiones HTTP eficientes con **Retrofit / Ktor**, serialización JSON y base de datos local con **Room**.

### 3. Mobile-to-Backend Integration
- **Seguridad en Mobile:** Autenticación mediante **JWT** / **OAuth2**, almacenamiento seguro de tokens (*EncryptedSharedPreferences*) y HTTPS Pinning.
- **Servicios en Segundo Plano:** Tareas diferidas y persistentes con **WorkManager** e integración de notificaciones Push (**Firebase Cloud Messaging - FCM**).
- **Estrategias Offline-First:** Sincronización de datos entre Room y la API Backend con estrategias de caché local.

---

## 🛠️ Tech Stack & Herramientas
- **Lenguaje:** Kotlin
- **UI Framework:** Jetpack Compose (Material Design 3)
- **Arquitectura:** Clean Architecture + MVVM
- **Asincronía & Reactividad:** Kotlin Coroutines & StateFlow
- **Inyección de Dependencias:** Hilt / Koin
- **Networking & BD:** Ktor / Retrofit, Room Database
