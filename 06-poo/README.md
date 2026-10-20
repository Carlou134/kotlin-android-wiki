# Programación orientada a objetos

En esta carpeta aprendes a **modelar cosas del mundo real** juntando datos y comportamiento en clases: cómo crearlas, cómo proteger sus propiedades, dos tipos especiales que Kotlin te da listos (**data classes** y **enums**) y cómo esconder lo interno con **visibilidad**.

-----

## Antes de empezar

Conviene que ya hayas leído [Funciones](../05-funciones/README.md). Una clase agrupa propiedades y funciones, y sus funciones siguen las mismas reglas que ya conoces.

Si en algún momento aparece una palabra que no conoces, búscala en el [Glosario](Glosario.md).

-----

## Orden de lectura

Las lecciones se apoyan una en la otra, así que conviene leerlas en este orden:

| Lección | Qué aprendes | Qué necesitas saber antes |
| --- | --- | --- |
| [1. Clases](01-clases.md) | Crear clases e instancias, constructor primario, bloque `init` y funciones miembro | Funciones |
| [2. Getters y setters](02-getters-y-setters.md) | Setters y getters personalizados, `field` y propiedades calculadas | Clases |
| [3. Data classes](03-data-classes.md) | Ventajas sobre clases normales: `==`, `copy()`, desestructuración, `hashCode()` y cuándo usarlas | Clases |
| [4. Enums](04-enums.md) | Declarar enums, usarlos con `when`, propiedades, `entries` y conversión desde números | Clases y `when` |
| [5. Visibilidad y encapsulamiento](05-visibilidad-y-encapsulamiento.md) | `public`, `private`, `protected`, `internal`, `private set` y por qué `private` no es seguridad | Clases y getters/setters |

-----

## El mapa completo en una mirada

```
Clases             ->  "el plano de un objeto"
   class Car(val model: String, val year: Int) { init { ... }  fun drive() {...} }
   val auto = Car("Tesla", 2024)     auto.model   auto.drive()

Getters y setters  ->  "controlo qué se lee y qué se guarda"
   var email = ""  set(value) { field = value.trim() }
   val esValido get() = email.contains("@")

Data classes       ->  "clases que solo guardan datos"
   data class Email(...)  ->  toString, ==, copy(), (a, b) = email

Enums              ->  "un conjunto fijo de opciones"
   enum class Estado { NUEVO, LEIDO }   when (estado) { ... }   entries

Visibilidad        ->  "escondo lo de adentro, expongo lo necesario"
   class Cuenta(private var saldo: Double) { fun depositar(m: Double) { ... } }
   var saldo = 0.0  private set
```

-----

## Cómo está armada cada lección

1. **En una frase:** qué es, sin jerga.
2. **Antes de empezar:** qué tienes que saber y las palabras nuevas.
3. **El contenido:** la explicación con ejemplos.
4. **Resumen en 5 líneas:** para repasar.
5. **Siguiente lección:** para seguir el orden.

-----

## Después de esta carpeta

Ya sabes modelar objetos. El último paso de los fundamentos es manejar datos que faltan y errores: [Null safety y errores](../07-null-safety-y-errores/README.md).
