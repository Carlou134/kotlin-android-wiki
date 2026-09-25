# Using Text Variables

## Strings & Characters

En esta lección, seguiremos trabajando con **variables** y profundizaremos en los **tipos de datos** `String` (cadena) y `Char` (carácter).

![string](../../Images/PHP_m2l1e1.svg)

Recuerda que un `String` en Kotlin es una secuencia de caracteres envuelta entre comillas dobles. Sus caracteres pueden incluir diversas combinaciones de dígitos, letras, símbolos y espacios en blanco.

```kotlin
var nombreVariable: String = "algún texto"
```

Los `String` están formados por caracteres y, en Kotlin, un carácter es un tipo de dato que representa una sola letra, dígito o símbolo envuelto entre comillas simples. Se denota con el tipo `Char`.

```kotlin
var nombreVariable: Char = 'A'
```

Algunas cosas a tener en cuenta sobre los caracteres:

- El carácter `'A'` no es lo mismo que `'a'`, ya que estas letras representan diferentes Unicodes y el compilador no las reconocerá como equivalentes.

- El carácter `'1'` no es lo mismo que el número entero `1`, ya que estos valores representan dos tipos de datos diferentes: `Char` e `Int`.

Consulta la documentación de Kotlin para aprender más sobre `String` y `Char`.

-----

## String Concatenation

Los `String` en Kotlin pueden estar solos o combinarse con otros `String` en un proceso conocido como **concatenación de cadenas**. Con el operador más (`+`), podemos concatenar o unir un `String` a otro, dando como resultado una nueva palabra, frase u oración.

Observa el siguiente ejemplo:

```kotlin
var firstName = "Mike" 
var lastName = "Wazowski" 

println(firstName + " " + lastName) // Imprime: Mike Wazowski
```

En el código anterior, hemos combinado las variables `firstName` y `lastName` para producir un nombre completo usando el operador `+`. Observa cómo hemos agregado un `String` vacío con un espacio (`" "`) entre las dos variables para crear un espacio natural entre las dos palabras.

Además de mostrar el resultado de un `String` concatenado, también podemos almacenar este proceso en una variable para usarlo más tarde. Por ejemplo:

```kotlin
val userGreeting = firstName + " " + lastName + ", how are you doing today?" 

println(userGreeting) 
```

El compilador reemplaza cada variable con su valor y muestra el `String` final:

```
Mike Wazowski, how are you doing today?
```

----

## String Templates

Una alternativa a la concatenación de cadenas es una sintaxis conocida como **plantillas de cadena (String templates)**. Muchos desarrolladores prefieren las plantillas de cadena sobre la concatenación por su concisión y eficiencia. Observa la diferencia:

- **Concatenación:** `"Hello, " + variableName + "!"`
- **Plantilla:** `"Hello, $variableName!"`

Las plantillas de cadena contienen un `String` con cualquier nombre de variable precedido por el símbolo `$`. Observa que, a diferencia de la concatenación, no necesitamos especificar los espacios en blanco, lo que ahorra pulsaciones de teclado y limita la cantidad de código que escribimos.

Supongamos que estamos construyendo el siguiente programa para una empresa de viajes que proporciona a un usuario su itinerario:

```kotlin
val passengerName = "Alex" 
val trainNumber = 2039
val timeOfDeparture = "16:00"
val timeOfArrival = "20:00" 

println("Mrs. $passengerName will be on the $trainNumber train departing at $timeOfDeparture to DevCity and arriving at $timeOfArrival.")
```

Cada variable se evalúa dentro del texto produciendo la información final del itinerario:

```
Mrs. Alex will be on the 2039 train departing at 16:00 to DevCity and arriving at 20:00.
```

Las plantillas de cadena son ampliamente utilizadas y suelen ser la sintaxis preferida por los desarrolladores de Kotlin. Ten en cuenta que la concatenación de cadenas también es una estrategia válida, por lo que es importante familiarizarse con ambas cuando se colabora con otros.

----

## User Input

La entrada de usuario es una parte importante de las aplicaciones informáticas. Para desbloquear nuestros teléfonos, ingresamos un código. Cuando compramos en línea, proporcionamos nuestra información de pago. Nuestra entrada ayuda al funcionamiento de muchas aplicaciones que usamos a diario.

La función `readLine()` en Kotlin se usa para indicarle al programa que se detenga hasta que el usuario ingrese algún tipo de entrada. Una vez que se presiona la tecla Enter, cualquier cosa escrita en la consola es devuelta por la función `readLine()` como un `String`. Luego podemos guardar la entrada en una variable. Veamos `readLine()` en acción:

```kotlin
println("Por favor, ingresa tu color favorito.")
val myColor = readLine()
```

Las dos declaraciones anteriores se pueden dividir en 4 partes:

1. Una declaración `println()` para pedirle al usuario que responda una pregunta.
2. El nombre de nuestra variable inmutable, `myColor`.
3. El operador de asignación, `=`.
4. La función `readLine()`.

El programa le pedirá al usuario con una pregunta, aceptará su entrada con la función `readLine()` y devolverá la entrada `String` una vez que se presione la tecla Enter. El `String` devuelto inicializará la variable declarada `myColor`.

Ahora podemos usar una plantilla de cadena para mostrar la entrada del usuario contenida en la variable:

```kotlin
println("Tu color favorito es $myColor")
```

Antes de continuar, notemos un cambio en nuestro entorno actual. Para poder trabajar con la entrada de usuario, la terminal de salida en el lado derecho de nuestra pantalla ha sido cambiada a una terminal que ejecuta el shell Bash. Usamos esta nueva terminal para escribir la entrada en nuestro programa, así como para compilarlo y ejecutarlo.

Para ejecutar nuestro programa en este ejercicio, seguiremos estos 3 pasos:

1. Haz clic en el botón **Run** cuando el código esté completo.
2. Usa el siguiente comando en la terminal:
   ```bash
   kotlinc myCode.kt -include-runtime -d myCode.jar
   ```
3. Sigue con este último comando:
   ```bash
   java -jar myCode.jar
   ```

El primer paso asegura que el archivo Kotlin esté guardado. El segundo paso compila el archivo Kotlin (`myCode.kt`) a bytecode y lo guarda en un archivo `.jar`. El tercer paso ejecuta la aplicación compilada. Este proceso se describe con más detalle en este Tutorial de Kotlin.

-----

## Built-in Properties and Functions

Kotlin contiene propiedades y funciones integradas que se pueden usar con sus diversos tipos de datos. Veremos las propiedades y funciones integradas más comunes para variables de tipo `String`. Puedes acceder a la lista completa en la documentación de Kotlin.

En resumen, una **propiedad** proporciona información sobre un valor particular. Kotlin admite una única propiedad para `String`: `length`, que devuelve el número de caracteres en un `String`. Veamos un ejemplo:

```kotlin
val tallestMountain = "Mount Everest" 
print(tallestMountain.length) // Imprime: 13
```

El número de caracteres en `"Mount Everest"` es 13. Observa cómo el espacio en blanco también se incluye en el conteo.

Además, un tipo de dato `String` también contiene **funciones** integradas que se pueden usar para realizar ciertas acciones sobre el `String` al que se le aplican. Por ejemplo, la función `capitalize()` devuelve un `String` capitalizado (primera letra mayúscula):

```kotlin
var name = "codecademy" 
println(name.capitalize()) // Imprime: Codecademy 
```

Ten en cuenta que, aunque el valor devuelto es `Codecademy`, el `String` original en minúsculas `"codecademy"` permanece sin cambios:

```kotlin
println(name) // Imprime: codecademy
```

Para mantener cualquier cambio hecho por la función, podemos reasignar el resultado de `name.capitalize()` a `name` y actualizar la variable:

```kotlin
name = name.capitalize()
println(name) // Imprime: Codecademy 
```

O podemos almacenar el resultado en una variable completamente nueva:

```kotlin
var capitalizedName = name.capitalize()
println(capitalizedName) // Imprime: Codecademy
```

**Nota:** Los caracteres (`Char`) tienen sus propias propiedades y funciones integradas en Kotlin. Consulta esta página en la documentación de Kotlin para ver la lista completa.

-----

## Character Escape Sequences

En Kotlin, tenemos la capacidad de formatear nuestro texto con la ayuda de **secuencias de escape**. Cuando se escapan, caracteres específicos como `t` y `n` representan una acción o comando que el compilador debe ejecutar.

Observa algunas de las siguientes secuencias de escape y sus operaciones:

- `\n` → Inserta una nueva línea
- `\t` → Inserta un tabulador
- `\r` → Inserta un retorno de carro
- `\'` → Inserta una comilla simple
- `\"` → Inserta una comilla doble
- `\\` → Inserta una barra invertida (backslash)
- `\$` → Inserta el símbolo de dólar

Cuando se usan en texto, cada barra invertida (`\`) y el carácter que la sigue serán reconocidos y evaluados por el compilador. Se pueden usar múltiples secuencias de escape en un solo `String`.

Una de las secuencias de escape más utilizadas es `\"`, ya que nos permite citar frases entre comillas dentro de un `String`. Observa:

```kotlin
print("\"May the Force be with you.\" - Star Wars, 1977")
```

En la terminal de salida veríamos:

```
"May the Force be with you." - Star Wars, 1977
```

Intenta el desafío de secuencias de escape que aparece a continuación para perfeccionar tu comprensión de estos comportamientos.

----

## Review

¡Buen trabajo! Has completado la segunda lección sobre **variables** y has aprendido los fundamentos del texto en Kotlin. Repasemos lo que vimos.

- Los **String** son secuencias de caracteres envueltas entre comillas dobles.
- Un **Carácter (`Char`)** representa un solo dígito, símbolo o letra envuelto entre comillas simples.
- La **concatenación de cadenas** es el proceso de unir `String` usando el operador `+`.
- Las **plantillas de cadena** se usan para sustituir valores de variables dentro de un `String` usando el símbolo `$`.
- La función `readLine()` se usa para aceptar la entrada de un usuario a través del shell Bash.
- La propiedad integrada `length` determina el número de caracteres en un `String`.
- La función integrada `capitalize()` se usa para capitalizar (poner la primera letra en mayúscula) el `String` sobre el cual se aplica.
- Las **secuencias de escape** permiten que el compilador realice ciertas operaciones de formato en un `String`.

Ejecuta el código en `Review.kt` y observa el mensaje de Codey. Siéntete libre de usar también este espacio para perfeccionar tu comprensión de las variables de texto y practicar la escritura de código en Kotlin.

-----