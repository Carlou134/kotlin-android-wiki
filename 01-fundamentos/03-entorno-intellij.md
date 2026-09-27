# Instalación y configuración de IntelliJ IDEA para Kotlin

## En una frase

IntelliJ IDEA es el IDE donde creas, ejecutas y depuras tus proyectos Kotlin sin tener que compilar a mano.

-----

## Antes de empezar

Conviene que ya sepas:

* Qué hace el IDE por ti cuando presionas Run: [Del editor a la salida](02-del-editor-a-la-salida.md).

Palabras nuevas (también están en el [Glosario](Glosario.md)):

* **Breakpoint:** Punto del código donde el programa se pausa en modo Debug para que inspecciones las variables.
* **Gradle:** Herramienta que gestiona las dependencias y la compilación de un proyecto.
* **IDE:** Entorno de desarrollo: editor con herramientas para escribir, ejecutar y depurar código.
* **JDK:** Kit de desarrollo de Java. Todo proyecto Kotlin para la JVM necesita uno para compilar y ejecutar.

-----

## ¿Cómo empezar con Kotlin en IntelliJ IDEA paso a paso?

### 1. Instalar IntelliJ IDEA (versión gratis)

- Ve a la página oficial de JetBrains.
- Descarga **IntelliJ IDEA Community Edition** (es gratuita).
- Instálalo según tu sistema operativo (Mac o Windows).
- Ábrelo y acepta los avisos de seguridad si aparecen.

---

### 2. Configurar Kotlin en un nuevo proyecto

- Cada proyecto de Kotlin necesita una versión de Java (JDK).  
- Elige una reciente como **Amazon Corretto 21** (el mismo IDE te ayuda a descargarla).
- El proyecto puede usar **Gradle** para manejar dependencias (el IDE crea las carpetas necesarias automáticamente).

---

### 3. Personalizar con plugins (opcional pero útil)

Puedes instalar plugins para agregar funciones extras, como:
- Docker
- GitHub Copilot
- Lombok
- Spring Boot Assistance

Estos se instalan desde la sección de **Plugins** dentro del IDE.

---

### 4. Mejorar la apariencia del editor

- Cambia el **tamaño de fuente** (ejemplo: 23 puntos).
- Ajusta el **interlineado**.
- Modifica los **colores** para que sea más fácil de leer.

Esto ayuda a reducir la fatiga visual.

---

### 5. Depurar (debug) tu código

El debug te permite ejecutar el código línea por línea para encontrar errores.

**Pasos básicos:**
1. Crea un archivo Kotlin (ej: `main.kt`).
2. Agrega variables y `println`.
3. Pon **breakpoints** (pausas) haciendo clic en el margen izquierdo.
4. Ejecuta en **modo Debug** (ícono de insecto 🐞).
5. Observa cómo cambian las variables paso a paso.

---

### 6. Configuraciones principales en "Settings"

- **Appearance and Behavior**: cambiar la interfaz y temas.
- **Editor**: ajustar autoimports, atajos de teclado, fuente, tamaño.
- **Plugins**: añadir o quitar extensiones.
- **Version Control**: conectar Git, GitHub o GitLab.
- **Build, Execution, Deployment**: configurar Gradle, Maven o la versión de Java (JVM).

---

## Resumen final

Para empezar con Kotlin de forma sencilla:
1. Instala IntelliJ IDEA Community Edition.
2. Crea un proyecto con JDK (ej: Amazon Corretto 21).
3. Personaliza plugins, fuente y colores a tu gusto.
4. Usa el **modo Debug** para revisar tu código paso a paso.
5. Ajusta configuraciones según necesites (Git, Gradle, etc.).

-------

## Creación de proyecto Kotlin en IntelliJ y primer programa

## Cómo crear tu primer proyecto en Kotlin con IntelliJ IDEA

### 1. Crear un nuevo proyecto

- Abre IntelliJ y selecciona **New Project**.
- Elige **Kotlin** como lenguaje.
- Ponle un nombre a tu proyecto (ej: "Kotlin Course dos").
- Selecciona la ubicación en tu computadora.
- Recomendaciones:
  - Usa la opción por defecto para el sistema de build.
  - Mantén Kotlin como lenguaje para Gradle DSL.
  - No generar código de ejemplo (para aprender desde cero).
  - Usa **JDK versión 17 o posterior**.

---

### 2. Estructura del proyecto

Un proyecto Kotlin tiene estas carpetas:

- **source**: todo el código fuente.
- **main**: código principal.
- **test**: para pruebas (no lo usaremos al inicio).

Dentro de `main`, crea una carpeta (ej: "clase tres") y dentro de ella tu primer archivo Kotlin (ej: `03_hello_world.kt`).

---

### 3. Escribir tu primer código

Todo programa Kotlin comienza con `fun main()`. El código dentro de las llaves `{}` se ejecuta línea por línea.

```kotlin
fun main() {
    println("Bienvenido a tu primer programa en Kotlin")
}
```

Para ejecutarlo, haz clic en la **flecha verde** junto a `fun main`.

---

### 4. Diferencia entre `print` y `println`

- **`println`**: imprime un mensaje y **agrega un salto de línea** al final.
- **`print`**: imprime un mensaje **sin salto de línea**.

**Ejemplo:**

```kotlin
print("Sistema iniciado...")
println("¡Listo!")
```

**Salida en consola:**

```
Sistema iniciado...¡Listo!
```

---

### 5. Comentarios en Kotlin

Los comentarios son texto que Kotlin ignora. Sirven para explicar o deshabilitar código.

- **Una línea:**  
  `// Esto es un comentario`

- **Múltiples líneas:**  
  ```kotlin
  /* Esto es un comentario
     de múltiples líneas */
  ```

---

### 6. Ejercicio práctico para practicar

Crea un programa que:

1. Muestre tu nombre.
2. Muestre el nombre de una aplicación de email.
3. Muestre un mensaje motivacional.
4. Use al menos dos `println`.

**Ejemplo:**

```kotlin
println("Hola, soy Juan")
println("Bienvenido a Gmail")
println("¡Vamos a aprender Kotlin juntos!")
println("El futuro es ahora")
```

Ejecuta el código usando el botón **Run** (flecha verde) en la esquina superior derecha.

---

## Resumen final

1. Crea un proyecto Kotlin con JDK 17+.
2. Escribe tu código dentro de `fun main()`.
3. Usa `println` para imprimir con salto de línea, o `print` sin salto.
4. Agrega comentarios con `//` o `/* */`.
5. Ejecuta frecuentemente para ver los resultados en consola.

-----

## Resumen en 5 líneas

1. Instala IntelliJ IDEA Community Edition, que es gratuita.
2. Cada proyecto necesita un JDK (17 o posterior); el propio IDE puede descargarlo.
3. Tu código va dentro de `fun main()` y se ejecuta con la flecha verde (Run).
4. El modo Debug con breakpoints te deja avanzar línea por línea y ver cómo cambian las variables.
5. En Settings ajustas apariencia, plugins, Git y la configuración de Gradle o de la JVM.

-----

## Siguiente lección

Terminaste esta carpeta. Sigue con la siguiente: [Tipos y variables](../02-tipos-y-variables/README.md).
