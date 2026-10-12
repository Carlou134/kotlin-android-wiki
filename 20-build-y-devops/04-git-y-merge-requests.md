# Git y merge requests

## En una frase

Trabajar bien con Git es dejar una **historia que se entiende**: commits chicos que hacen una sola cosa y explican por qué, ramas cortas que salen de `main` y vuelven pronto, y merge requests fáciles de revisar.

-----

## Antes de empezar

No necesitas conocimientos previos de esta wiki, solo los comandos básicos de Git (`add`, `commit`, `push`).

Palabras nuevas (también están en el [Glosario](Glosario.md)):

* **Commit atómico:** Un commit que contiene un solo cambio lógico, completo y que compila.
* **Conventional Commits:** Convención para los mensajes de commit: `tipo(alcance): descripción`.
* **Rama (branch):** Línea de trabajo paralela que sale de otra (normalmente `main`).
* **Rebase:** Reaplicar tus commits encima de otra base, reescribiendo su historia.
* **Merge request (MR):** En GitLab, la propuesta de integrar una rama en otra, con revisión y CI.
* **Conflicto:** Cuando dos ramas cambiaron las mismas líneas y Git no puede decidir solo.

-----

## Commits atómicos

Un commit atómico:

* Hace **una sola cosa**: agregar una funcionalidad, corregir un bug, renombrar algo.
* Está **completo**: el proyecto compila y los tests pasan en ese commit.
* Se puede **revertir** solo, sin arrastrar cambios que no tienen que ver.

```
❌ "cambios varios"   (agrega un endpoint, corrige un bug no relacionado y reformatea 30 archivos)

✅ refactor(pedidos): extraer el cálculo de impuestos a una función
✅ feat(pedidos): permitir cancelar pedidos pendientes
✅ test(pedidos): cubrir la cancelación de pedidos ya enviados
```

Si un bug aparece en producción, una historia así te dice **qué** commit lo introdujo, y lo puedes revertir sin perder lo demás.

Para armar commits atómicos aunque hayas tocado varias cosas a la vez, `git add -p` te deja elegir **qué partes** de cada archivo van en el commit.

## Conventional Commits

```
tipo(alcance): descripción en imperativo

Cuerpo opcional: el POR QUÉ del cambio, no el qué (el qué ya está en el diff).
```

| Tipo | Para |
| --- | --- |
| `feat` | Una funcionalidad nueva |
| `fix` | Un bug corregido |
| `refactor` | Cambio de estructura sin cambiar el comportamiento |
| `test` | Agregar o corregir tests |
| `docs` | Documentación |
| `chore` | Mantenimiento: dependencias, configuración |
| `ci` | El pipeline |
| `perf` | Mejora de rendimiento |

Ejemplo con cuerpo:

```
fix(sincronizacion): no pisar datos nuevos con respuestas atrasadas

El proveedor a veces responde fuera de orden. El upsert ahora solo
actualiza si el timestamp recibido es más reciente que el guardado.
```

El cuerpo explica la **razón**. Dentro de 6 meses, nadie va a recordar por qué existe esa condición en el `WHERE`; el commit sí.

## Ramas cortas

```
git switch main
git pull
git switch -c feat/cancelar-pedidos
```

* Cada cambio en su rama, que sale de `main` actualizado.
* Nombres con prefijo: `feat/...`, `fix/...`, `refactor/...`.
* **Ramas cortas:** días, no semanas. Cuanto más vive una rama, más se aparta de `main` y más conflictos acumula.

## Mantenerse al día: rebase

Mientras trabajas, `main` avanza. Antes de abrir el MR (o si hay conflictos), trae esos cambios:

```
git fetch
git rebase origin/main
```

`rebase` toma tus commits y los reaplica **encima** del `main` actual. La historia queda lineal, como si hubieras empezado hoy.

Si hay conflictos, Git se detiene en cada commit:

```
# editar los archivos en conflicto y dejarlos como deben quedar
git add <archivos>
git rebase --continue      # o git rebase --abort para volver a como estabas
```

Como el rebase **reescribe** tus commits, el push necesita forzarse:

```
git push --force-with-lease
```

`--force-with-lease` falla si alguien más subió algo a tu rama mientras tanto, en lugar de pisarlo. **Nunca** uses `--force` a secas, y **nunca** hagas rebase de una rama que comparten otras personas, como `main`.

## Antes de abrir el MR: limpiar la historia

Mientras trabajas, aparecen commits como "wip", "arreglo typo" o "ahora sí". Antes de pedir revisión, puedes reordenarlos y juntarlos con un rebase interactivo (`git rebase -i origin/main`), para que cada commit del MR cuente una parte de la historia.

Otra opción es que GitLab haga **squash** al mergear: todos los commits del MR se convierten en uno. Es más simple, pero pierdes la separación entre, por ejemplo, el refactor y la funcionalidad. Sigue la convención del equipo.

## El merge request

Un buen MR:

* **Es chico.** 200 líneas se revisan con atención; 2.000 se aprueban sin leer. Si una funcionalidad es grande, divídela: primero el refactor, después el modelo, después la ruta.
* **Tiene una descripción útil:**

```markdown
## Qué
Permite cancelar pedidos en estado PENDIENTE desde la API.

## Por qué
Soporte cancela pedidos a mano en la base (ticket #123).

## Cómo probar
1. `docker compose up -d`
2. Crear un pedido y llamar a `POST /api/v1/pedidos/{id}/cancelar`
3. Un pedido ENVIADO devuelve 409

## Notas
Agrega la migración V012 (columna `motivo_cancelacion`, nullable).
```

* **Pasa el CI antes de pedir revisión.** Nadie debería revisar código que no compila.
* **Usa Draft** mientras no está listo, para mostrar avances sin pedir aprobación.

## Revisar y ser revisado

**Al revisar:** comenta sobre el código, no sobre la persona; distingue lo que bloquea ("esto filtra datos entre empresas") de las sugerencias ("opcional: podría ser una extensión"); y pregunta cuando no entiendes, en vez de suponer.

**Al ser revisado:** responde cada comentario (aunque sea "hecho" o "lo dejo así porque..."), no lo tomes personal, y si una discusión se alarga, hablala en persona.

## Lo que nunca va al repositorio

`.gitignore` en la raíz:

```
build/
.gradle/
.idea/
*.iml
.env
```

**Nunca** subas secretos (contraseñas, API keys, claves privadas). Si se te escapa uno, borrarlo en un commit nuevo **no alcanza**: sigue en la historia. Hay que **rotarlo** (generar uno nuevo e invalidar el viejo) de inmediato.

## Comandos del día a día

```
git status                       # qué cambió
git diff                         # los cambios sin agregar
git add -p                       # agregar por partes
git commit                       # con mensaje de Conventional Commits
git switch -c feat/x             # rama nueva
git fetch && git rebase origin/main
git push --force-with-lease      # después de un rebase
git stash / git stash pop        # guardar cambios a medias y recuperarlos
git log --oneline --graph        # la historia, compacta
```

-----

## Resumen en 5 líneas

1. Commits atómicos: un cambio lógico, completo y que compila; `git add -p` ayuda a separarlos.
2. Conventional Commits (`feat`, `fix`, `refactor`...) con un cuerpo que explica el por qué.
3. Ramas cortas desde `main`; actualízalas con `git rebase origin/main` y sube con `--force-with-lease`, nunca en ramas compartidas.
4. MRs chicos, con qué, por qué y cómo probar, en verde en el CI antes de pedir revisión.
5. Nada de secretos en el repositorio; si se escapa uno, rotarlo, porque borrarlo no alcanza.

-----

## Siguiente lección

[5. GitLab CI](05-gitlab-ci.md): la verificación automática de cada cambio.
