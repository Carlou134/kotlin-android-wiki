# Glosario

Las palabras que aparecen en las lecciones de esta carpeta, explicadas de forma simple. Están en orden alfabético.

-----

**Abstracción.** Una interfaz que describe *qué* se hace sin decir *cómo*.

**Acoplamiento.** Cuánto depende una pieza de otra. Mucho acoplamiento significa que si cambias una, se rompe la otra.

**Adapter.** Implementación concreta que conecta un puerto con una tecnología (Ktor, Exposed, un cliente HTTP).

**Adapter primario (driving).** Adapter que *llama* al núcleo, como una ruta HTTP.

**Adapter secundario (driven).** Adapter que *es llamado* por el núcleo, como un repositorio de base de datos.

**Anti-Corruption Layer (ACL).** Capa que traduce el modelo de un sistema externo al tuyo, para que el ajeno no contamine tu dominio.

**Bounded context.** Una parte del negocio con límites claros, donde cada palabra significa una sola cosa.

**Command.** Data class con los datos de entrada de un use case.

**Composition root.** Punto de arranque donde se crean y conectan las piezas concretas. Es el único lugar que las conoce todas.

**Contenedor de DI.** Librería que crea los objetos y resuelve sus dependencias por ti, como Koin o Hilt.

**Detalle.** La implementación concreta: la base de datos, el framework web, el cliente HTTP.

**DTO.** Objeto que solo transporta datos hacia afuera o desde afuera (JSON, filas de tabla).

**Error de negocio.** Un resultado esperado que no es el feliz ("sin stock"), a diferencia de un fallo técnico.

**Inversión de dependencias (DIP).** Principio por el que lo importante define una interfaz y lo concreto la implementa.

**Inyección de dependencias (DI).** Pasarle a una clase lo que necesita desde afuera, en vez de que lo cree ella.

**Koin.** Contenedor de DI liviano para Kotlin, sin generación de código.

**Konsist.** Librería de Kotlin para escribir tests de arquitectura y de convenciones.

**Lenguaje ubicuo.** El vocabulario que comparten el código y la gente de negocio dentro de un bounded context.

**Monolito modular.** Una sola aplicación desplegable, dividida internamente en módulos que no se pisan.

**Outcome.** Tipo `sealed` con todos los resultados posibles de un use case.

**Puerto (port).** Interfaz que define cómo se habla con el núcleo. El de entrada dice qué puede pedirle el mundo; el de salida, qué necesita el núcleo del mundo.

**Regla de dependencia.** Las dependencias siempre apuntan hacia adentro: infrastructure → application → domain.

**Scope (Konsist).** El conjunto de archivos que Konsist analiza.

**Service locator.** Antipatrón en el que una clase le pide sus dependencias al contenedor en vez de recibirlas por constructor.

**Shared kernel.** Módulo chico con lo que varios contextos comparten de verdad (IDs, tipos base).

**SOLID.** Cinco principios de diseño: responsabilidad única, abierto/cerrado, sustitución de Liskov, segregación de interfaces e inversión de dependencias.

**Test de arquitectura.** Test que inspecciona la estructura del código: paquetes, imports, nombres, anotaciones.

**Use case.** Una operación de negocio completa, como "crear pedido" o "cancelar pedido".
