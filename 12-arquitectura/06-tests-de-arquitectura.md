# Tests de arquitectura con Konsist

## En una frase

Un test de arquitectura revisa el **código fuente** (no su comportamiento) y falla si alguien rompe una regla de diseño, como importar Exposed desde el dominio o nombrar mal un use case.

-----

## Antes de empezar

Conviene que ya sepas:

* Las capas y la regla de dependencia: [Arquitectura hexagonal](02-arquitectura-hexagonal.md).
* Las convenciones de nombres: [La capa application](05-capa-application.md).
* Lo básico de un test con Kotest (se ve en `19-testing`).

Palabras nuevas (también están en el [Glosario](Glosario.md)):

* **Test de arquitectura:** Test que inspecciona la estructura del código: paquetes, imports, nombres, anotaciones.
* **Konsist:** Librería de Kotlin para escribir tests de arquitectura y de convenciones.
* **Scope:** El conjunto de archivos que Konsist analiza.

-----

## El problema: las reglas que se olvidan

Un equipo acuerda "el dominio no importa frameworks". Seis meses después alguien, apurado, agrega `@Serializable` a una entidad de dominio. El code review no lo ve. La regla murió.

Los módulos de Gradle protegen **algunos** límites (lo que un módulo no declara, no lo puede importar). Pero dentro de un módulo, `domain` e `infrastructure` son solo paquetes, y nada impide que uno importe al otro.

Los tests de arquitectura cierran ese hueco: **la regla se ejecuta en cada build**.

## Regla de capas

```kotlin
class ArquitecturaTest : FunSpec({

  test("las capas respetan la regla de dependencia") {
    Konsist
      .scopeFromProduction()
      .assertArchitecture {
        val domain = Layer("Domain", "com.tienda..domain..")
        val application = Layer("Application", "com.tienda..application..")
        val infrastructure = Layer("Infrastructure", "com.tienda..infrastructure..")

        domain.dependsOnNothing()
        application.dependsOn(domain)
        infrastructure.dependsOn(application, domain)
      }
  }
})
```

Si alguien importa algo de `infrastructure` desde `domain`, el test falla y te dice **qué archivo** rompió la regla.

## Reglas de convenciones

Konsist también verifica nombres y ubicaciones:

```kotlin
test("los use cases viven en application") {
  Konsist
    .scopeFromProduction()
    .classes()
    .withNameEndingWith("Service")
    .assertTrue { it.resideInPackage("..application..") }
}

test("el dominio no usa anotaciones de serialización") {
  Konsist
    .scopeFromProduction()
    .classes()
    .withPackage("..domain..")
    .assertFalse { it.hasAnnotationWithName("Serializable") }
}

test("los DTOs de la API terminan en Request o Response") {
  Konsist
    .scopeFromProduction()
    .classes()
    .withPackage("..infrastructure.web..")
    .withAnnotationNamed("Serializable")
    .assertTrue { it.name.endsWith("Request") || it.name.endsWith("Response") }
}
```

## Tests de contrato sobre las rutas

Otra regla útil para una API: "toda ruta declara explícitamente cómo se autentica". Se puede verificar de dos formas:

* **Con Konsist**, revisando que los archivos de rutas usen `authenticate { }` o una marca explícita de ruta pública.
* **En tiempo de ejecución**, levantando la app en un test y recorriendo el árbol de rutas registradas.

La idea es la misma: si alguien agrega una ruta y se olvida de la autenticación, el build falla antes de que llegue a producción.

## Qué conviene verificar (y qué no)

| Buen candidato | Mal candidato |
| --- | --- |
| Regla de dependencia entre capas | Gustos de estilo (para eso están ktlint y detekt) |
| Sufijos de nombres (`Service`, `Command`, `Outcome`) | Reglas con muchas excepciones |
| El dominio sin anotaciones de frameworks | Lógica de negocio (para eso están los tests normales) |
| Toda ruta con autenticación explícita | |

Una regla con muchas excepciones genera ruido y el equipo termina ignorándola. Es mejor tener pocas reglas y que siempre se cumplan.

-----

## Resumen en 5 líneas

1. Un test de arquitectura inspecciona la estructura del código, no su comportamiento.
2. Los módulos de Gradle protegen límites entre módulos; Konsist protege las capas dentro de cada módulo.
3. `assertArchitecture` con `Layer` y `dependsOn` verifica la regla de dependencia.
4. También sirve para convenciones: sufijos, paquetes, anotaciones prohibidas, rutas con autenticación.
5. Pocas reglas, sin excepciones, ejecutadas en cada build: así una decisión de diseño no se olvida.

-----

## Siguiente lección

Terminaste la arquitectura. Lo próximo es la puerta de entrada al sistema: `13-http-y-apis`. Vuelve al índice: [README principal](../README.md).
