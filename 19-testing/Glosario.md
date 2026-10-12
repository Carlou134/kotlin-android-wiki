# Glosario

Las palabras que aparecen en las lecciones de esta carpeta, explicadas de forma simple. Están en orden alfabético.

-----

**Análisis estático.** Revisar el código sin ejecutarlo.

**Arrange, Act, Assert.** Estructura de un test: preparar los datos, ejecutar la acción y verificar el resultado.

**Base en memoria (H2).** Base de datos que corre dentro de la JVM; rápida, pero no es PostgreSQL.

**Baseline.** Archivo con los problemas existentes que se aceptan, para exigir solo sobre el código nuevo.

**Cobertura (coverage).** Porcentaje del código que se ejecutó durante los tests.

**Cobertura de ramas (branch coverage).** Porcentaje de caminos de los `if` y `when` que se ejecutaron.

**Contenedor.** Un proceso aislado con su propio sistema de archivos, creado a partir de una imagen de Docker.

**detekt.** Herramienta de análisis estático para Kotlin que detecta olores de código y errores comunes.

**Doble de prueba.** Cualquier objeto que reemplaza a uno real en un test.

**`.editorconfig`.** Archivo estándar con reglas de formato que leen los editores y ktlint.

**Fake.** Implementación simplificada pero funcional de un puerto, por ejemplo un repositorio en memoria.

**Kotest.** Framework de testing para Kotlin con specs legibles y matchers expresivos.

**Kover.** Herramienta de JetBrains para medir la cobertura de tests en Kotlin.

**ktlint.** Linter que aplica el estilo oficial de formato de Kotlin.

**Linter.** Herramienta que revisa el código fuente buscando problemas de estilo o errores.

**Matcher.** Función de aserción, como `shouldBe` o `shouldThrow`.

**Mock.** Doble que devuelve respuestas preparadas y registra cómo se lo llamó, para verificarlo después.

**MockK.** Librería de mocks para Kotlin, con soporte para funciones `suspend`.

**Par de claves de prueba.** Claves RSA generadas solo para los tests, para firmar tokens válidos.

**Pirámide de tests.** Muchos tests unitarios rápidos, algunos de integración y pocos end-to-end.

**Slot.** Contenedor de MockK que captura el argumento con el que se llamó a un mock.

**Spec.** Una clase de Kotest que agrupa tests (`FunSpec`, `DescribeSpec`, `BehaviorSpec`...).

**Stub.** Doble que devuelve respuestas preparadas.

**Test de contrato de API.** Test que verifica lo que ve el cliente: rutas, códigos de estado, JSON.

**Test de integración.** Test que prueba una pieza junto con tecnología real: una base de datos, un servidor.

**Test unitario.** Test que prueba una pieza aislada, sin base de datos ni red.

**`testApplication`.** Función de Ktor que levanta la aplicación en un motor de pruebas en memoria.

**Testcontainers.** Librería que crea y destruye contenedores de Docker desde el código de los tests.
