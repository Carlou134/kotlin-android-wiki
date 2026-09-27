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

Las próximas etapas continúan la numeración (`08-...`) siguiendo la hoja de ruta de abajo.

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
