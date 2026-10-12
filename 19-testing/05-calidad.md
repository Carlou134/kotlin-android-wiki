# Calidad: Kover, ktlint y detekt

## En una frase

Además de los tests, tres herramientas vigilan el código en cada build: **ktlint** (formato), **detekt** (problemas de diseño y errores comunes) y **Kover** (cuánto código ejecutan los tests); juntas evitan discusiones de estilo en el code review y detectan problemas antes de que lleguen a producción.

-----

## Antes de empezar

Conviene que ya sepas:

* Escribir tests: [Kotest](01-kotest.md).
* Tests de arquitectura con Konsist: [Tests de arquitectura](../12-arquitectura/06-tests-de-arquitectura.md).

Palabras nuevas (también están en el [Glosario](Glosario.md)):

* **Linter:** Herramienta que revisa el código fuente buscando problemas de estilo o errores.
* **Análisis estático:** Revisar el código sin ejecutarlo.
* **Cobertura (coverage):** Porcentaje del código que se ejecutó durante los tests.
* **Cobertura de ramas (branch coverage):** Porcentaje de caminos de los `if` y `when` que se ejecutaron.
* **Baseline:** Archivo con los problemas existentes que se aceptan, para exigir solo sobre el código nuevo.
* **`.editorconfig`:** Archivo estándar con reglas de formato (indentación, largo de línea) que leen los editores y ktlint.

-----

## ktlint: formato sin discusiones

¿Dos espacios o cuatro? ¿Coma al final de la lista? Son decisiones que no importan, **siempre que todos hagan lo mismo**. ktlint aplica el estilo oficial de Kotlin y termina la discusión.

Con el plugin de Gradle:

```
./gradlew ktlintCheck    # revisa y falla si algo no cumple
./gradlew ktlintFormat   # corrige automáticamente lo que puede
```

Las reglas se ajustan en `.editorconfig`, en la raíz del repositorio:

```ini
[*.{kt,kts}]
indent_size = 2
max_line_length = 120
ktlint_code_style = ktlint_official
```

El mismo archivo lo lee IntelliJ, así que el formateo del editor y el del build coinciden.

Un buen hábito: `ktlintFormat` antes de cada commit (incluso con un hook de git). En el code review nadie vuelve a comentar sobre espacios.

## detekt: problemas más allá del formato

detekt busca **olores de código** y errores comunes:

| Regla | Detecta |
| --- | --- |
| `LongMethod`, `LargeClass` | Funciones o clases demasiado grandes |
| `CyclomaticComplexMethod` | Demasiados caminos (`if`, `when`, bucles) en una función |
| `LongParameterList` | Funciones con demasiados parámetros |
| `TooGenericExceptionCaught` | `catch (e: Exception)` sin justificación |
| `MagicNumber` | Números sueltos sin nombre |
| `UnsafeCallOnNullableType` | Uso de `!!` |

```
./gradlew detekt
```

Se configura con un archivo propio que **ajusta** las reglas por defecto:

```kotlin
detekt {
  buildUponDefaultConfig = true                       // parte de la configuración por defecto
  config.setFrom("$rootDir/config/detekt/detekt.yml") // y aplica tus cambios
  baseline = file("$rootDir/config/detekt/baseline.xml")
}
```

### La baseline: adoptarlo en un proyecto existente

Si activas detekt en un proyecto con años de código, aparecen cientos de problemas. Arreglarlos todos antes de empezar no es realista.

`./gradlew detektBaseline` genera un archivo con los problemas **actuales**, que se ignoran. Desde ese momento, el build solo falla por problemas **nuevos**. La deuda vieja se paga de a poco; la nueva no entra.

### Suprimir con justificación

Cuando una regla no aplica en un caso concreto:

```kotlin
@Suppress("TooGenericExceptionCaught") // el job no debe morir por ningún error; se registra y sigue
catch (e: Exception) { ... }
```

Siempre con un comentario que explique **por qué**. Una supresión sin justificación es una regla ignorada.

## Kover: cobertura

Kover mide qué líneas y qué ramas ejecutaron los tests:

```
./gradlew koverHtmlReport   # reporte navegable en build/reports/kover
./gradlew koverVerify       # falla si no se alcanza el mínimo
```

```kotlin
kover {
  reports {
    verify {
      rule {
        minBound(80)   // al menos 80 % de líneas cubiertas
      }
    }
  }
}
```

### La cobertura es una señal, no un objetivo

```kotlin
test("cancelar") {
  service.ejecutar(command)   // ejecuta el código...
}                              // ...pero no verifica NADA
```

Este test sube la cobertura y no prueba nada. Si la cobertura se vuelve un objetivo, aparecen tests así.

Úsala para lo que sirve:

* **Encontrar código sin tests:** abre el reporte HTML y mira qué ramas de tus use cases están en rojo. ¿Hay un outcome sin test?
* **Evitar retrocesos:** un mínimo en `koverVerify` impide que la cobertura baje sin que nadie lo note.
* **Excluir lo que no aporta:** código generado, DTOs sin lógica, el `main`.

La cobertura de **ramas** es más útil que la de líneas: un `if` en una sola línea cuenta como cubierto aunque solo hayas probado uno de sus dos caminos.

## El orden en el pipeline

Del más barato al más caro, para fallar lo antes posible:

```
1. ktlintCheck       segundos    ¿está bien formateado?
2. detekt            segundos    ¿hay problemas de diseño?
3. test              minutos     unitarios + Konsist + testApplication
4. integrationTest   minutos     Testcontainers (necesita Docker)
5. koverVerify       segundos    ¿la cobertura alcanza el mínimo?
```

Si el formato está mal, no tiene sentido esperar 5 minutos de tests para enterarse. Lo armas en el CI en [GitLab CI](../20-build-y-devops/05-gitlab-ci.md).

**En tu máquina:** antes de subir cambios, corre al menos `ktlintFormat`, `detekt` y los tests del módulo que tocaste. Que el CI falle por formato es tiempo perdido para todo el equipo.

-----

## Resumen en 5 líneas

1. ktlint aplica el estilo oficial de formato (`ktlintCheck`, `ktlintFormat`), configurado en `.editorconfig`.
2. detekt detecta olores de código (funciones largas, complejidad, `catch` genéricos, `!!`); una baseline permite adoptarlo sin arreglar todo primero.
3. Las supresiones van siempre con un comentario que explique por qué.
4. Kover mide cobertura; úsala para encontrar ramas sin tests y evitar retrocesos, no como objetivo en sí.
5. En el pipeline, de lo más barato a lo más caro: formato, detekt, tests, integración y cobertura.

-----

## Siguiente lección

Terminaste testing y calidad. Vuelve al índice: [README principal](../README.md).
