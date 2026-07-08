# Actividad 0.3: Tu portafolio en GitHub

!!! warning "Descarga la plantilla"
    📄 [Plantilla 0.3 — Tu portafolio en GitHub](plantillas/Actividad_0_3_Plantilla.docx){target="_blank" rel="noopener"}

## Qué vas a practicar

En esta actividad crearás un repositorio **real y público** en tu cuenta de GitHub: tu portafolio de desarrollador. A lo largo del ejercicio simularás trabajar desde dos ordenadores distintos (el repositorio original y un clon en otra carpeta) y aprenderás a sincronizar cambios, resolver conflictos entre versiones y gestionar ramas en el servidor remoto.

Al terminar tendrás algo concreto en tu perfil de GitHub que podrás mostrar a futuros empleadores.

Aspectos que cubre la actividad:

- Crear un repositorio remoto vacío y enlazarlo con un repositorio local.
- Publicar cambios con `git push` y entender qué hace `-u`.
- Clonar un repositorio con `git clone` y configurar una identidad distinta.
- Distinguir `git fetch` de `git pull` observando el estado del grafo.
- Provocar y resolver un conflicto entre dos "máquinas" distintas.
- Publicar una rama en el remoto, hacer el merge y limpiar.

---

## Requisitos previos

- Cuenta de GitHub activa con acceso configurado (Token PAT o clave SSH, según lo que hayas hecho en la teoría).
- El alias `git lga` configurado:

```bash
git config --global alias.lga "log --graph --oneline --all --decorate"
```

- Comprueba que Git tiene tu nombre y correo reales:

```bash
git config --global user.name "Tu Nombre"
git config --global user.email "tuemail@ejemplo.com"
```

---

## Norma de la actividad

!!! warning "Obligatorio en cada paso"
    Antes de ejecutar cualquier comando que cambie el repositorio o interactúe con GitHub, escribe **qué esperas que pase**: qué verás en el grafo, si el push tendrá éxito o no, qué archivos cambiarán. Eso es lo que se valorará, no solo la captura del resultado.

---

## Preparación

Crea una carpeta `mi_portfolio` **fuera de OneDrive / iCloud / Google Drive** e inicializa el repositorio:

```bash
git init mi_portfolio
cd mi_portfolio
```

---

## Parte A — Primera puesta en marcha

### Paso 1 — El README inicial

Crea un archivo `README.md` dentro de `mi_portfolio` con la siguiente información sobre ti (redáctalo tú, no copies):

- Tu nombre y una frase que describa qué estás estudiando.
- Dos o tres tecnologías o herramientas que estés aprendiendo en el curso.
- Un objetivo profesional a corto plazo (una línea).

Haz el primer commit:

```bash
git add README.md
git commit -m "Añade README inicial del portafolio"
git lga
```

**Pregunta A.1:** ¿Cuántas referencias (etiquetas) aparecen en el grafo en este momento? Nómbralas.

### Paso 2 — Crear el repositorio remoto y enlazarlo

En GitHub, crea un repositorio **público** llamado `mi_portfolio`. Déjalo completamente vacío (sin README, sin .gitignore, sin licencia).

**Predice antes de ejecutar:** si enlazas el remoto y haces push ahora, ¿qué verás en la página de GitHub?

Enlaza y publica:

```bash
git remote add origin <URL-de-tu-repo>
git push -u origin main
git lga
```

**Preguntas A.2:**

1. ¿Qué hace exactamente el flag `-u` en este primer push? ¿Para qué sirve en los push siguientes?
2. Después del push, ¿qué etiqueta nueva aparece en el grafo que antes no estaba?
3. Comprueba en GitHub que el README se muestra en la portada del repositorio. ¿Por qué GitHub lo muestra automáticamente?

---

## Parte B — Segundo ordenador

### Paso 3 — Clonar el repositorio

Simula que llegas a otro ordenador. Clona el repositorio en una carpeta paralela llamada `mi_portfolio_clone` (desde **fuera** de `mi_portfolio`):

```bash
git clone <URL-de-tu-repo> mi_portfolio_clone
cd mi_portfolio_clone
git lga
```

**Pregunta B.1:** ¿Cuántas ramas aparecen en el grafo del clon? ¿Por qué aparece `origin/main` además de `main`?

Configura el clon para que sus commits parezcan hechos desde otro equipo:

```bash
git config user.name "Yo en casa"
git config user.email "yo.en.casa@ejemplo.com"
```

!!! note "Solo afecta a este repositorio"
    `git config` sin `--global` solo modifica la configuración del repositorio actual. Tu identidad global no cambia.

### Paso 4 — Añadir una sección desde el "segundo ordenador"

Desde `mi_portfolio_clone`, añade una nueva sección al final de `README.md`. Por ejemplo, una sección `## Proyectos en curso` con una o dos líneas describiendo en qué estás trabajando.

**Predice antes de ejecutar:** en este momento, ¿el repositorio original `mi_portfolio` sabe que este cambio existe? ¿Por qué?

```bash
git add README.md
git commit -m "Añade sección de proyectos en curso"
git push
git lga
```

Comprueba en GitHub que el nuevo commit aparece y que el autor figura como "Yo en casa".

---

## Parte C — Sincronización

### Paso 5 — Fetch: mirar antes de actuar

Vuelve a la carpeta `mi_portfolio` (el repositorio original).

**Predice antes de ejecutar:** ¿qué mostrará `git lga` ahora mismo en el repositorio original? ¿Sabe Git que hay un commit nuevo en el remoto?

```bash
git lga
```

Ahora descarga la información del remoto sin tocar tus archivos:

```bash
git fetch
git lga
```

**Preguntas C.1:**

1. ¿Ha cambiado algún archivo en tu carpeta después del `git fetch`?
2. ¿Qué ha cambiado en el grafo? ¿Qué etiqueta se ha movido?
3. ¿Dónde apunta `origin/main` ahora respecto a `main`?

### Paso 6 — Pull: integrar los cambios

**Predice antes de ejecutar:** ¿qué tipo de fusión hará Git al hacer `pull`? ¿Creará un commit de merge o moverá el puntero sin más?

```bash
git pull
git lga
```

**Preguntas C.2:**

1. ¿Ha sido un fast-forward o un merge commit? ¿Por qué?
2. Abre `README.md` en el repositorio original. ¿Está la sección que añadiste desde el clon?

---

## Parte D — Ramas divergentes y conflicto

Ahora ambas "máquinas" van a modificar el README al mismo tiempo.

### Paso 7 — Modificar desde el repositorio original

Desde `mi_portfolio`, añade una línea al final de `README.md`. Por ejemplo, una sección `## Contacto` con tu correo o tu perfil de LinkedIn.

```bash
git add README.md
git commit -m "Añade sección de contacto"
git push
git lga
```

### Paso 8 — Modificar desde el clon (conflicto inminente)

Desde `mi_portfolio_clone`, **sin hacer pull primero**, añade también una línea al final de `README.md`. Por ejemplo, una sección `## Idiomas` con los idiomas que hablas.

```bash
git add README.md
git commit -m "Añade sección de idiomas"
```

**Predice antes de ejecutar:** ¿tendrá éxito el siguiente push? Razona tu respuesta antes de ejecutarlo.

```bash
git push
```

Git te rechaza el push. Lee el mensaje de error.

**Preguntas D.1:**

1. Copia el mensaje de error y explica con tus palabras qué significa.
2. ¿Por qué `git push --force` sería peligroso en este momento? ¿Qué se perdería?

### Paso 9 — Resolver el conflicto

**Predice antes de ejecutar:** ¿en qué líneas de `README.md` esperas que aparezcan las marcas de conflicto?

```bash
git pull
```

Git no puede fusionar automáticamente. Abre `README.md` y localiza las marcas:

```
<<<<<<< HEAD
... (tu versión del clon)
=======
... (versión del remoto)
>>>>>>> origin/main
```

Edita el archivo para quedarte con **ambas secciones** (contacto e idiomas), borra las marcas y cierra el merge:

```bash
git add README.md
git commit -m "Resuelve conflicto: mantiene contacto e idiomas"
git push
git lga
```

**Preguntas D.2:**

1. ¿Qué representa cada una de las tres marcas (`<<<<<<<`, `=======`, `>>>>>>>`)?
2. Describe paso a paso cómo has resuelto el conflicto.
3. Comprueba en GitHub que el commit de merge aparece en el historial con los cambios de ambas partes.

---

## Parte E — Rama de nueva sección

### Paso 10 — Crear y publicar una rama

Vuelve a `mi_portfolio`. Actualiza primero:

```bash
git pull
git lga
```

Crea una rama para añadir una nueva sección al portafolio:

```bash
git switch -c seccion-habilidades
```

Añade una sección `## Habilidades` al `README.md` con tres habilidades técnicas que estés desarrollando. Haz commit:

```bash
git add README.md
git commit -m "Añade sección de habilidades"
```

**Predice antes de ejecutar:** después de este push, ¿cuántas ramas habrá en GitHub?

```bash
git push -u origin seccion-habilidades
git lga
```

Comprueba en GitHub que aparece el selector con las dos ramas: `main` y `seccion-habilidades`.

### Paso 11 — Fusionar y limpiar

Vuelve a `main` y fusiona la rama:

```bash
git switch main
```

**Predice antes de ejecutar:** ¿será fast-forward o necesitará merge commit? ¿Por qué?

```bash
git merge seccion-habilidades
git push
git lga
```

Limpia la rama local y la remota:

```bash
git branch -d seccion-habilidades
git push origin --delete seccion-habilidades
git lga
```

**Preguntas E.1:**

1. ¿Ha sido fast-forward? ¿Por qué sí o por qué no?
2. Después de borrar la rama remota, ¿siguen existiendo los commits que había en ella? ¿Dónde?
3. Comprueba en GitHub que solo queda la rama `main`.

---

## Tabla de decisión

Rellena esta tabla **sin ejecutar nada**. La columna "Por qué" es obligatoria:

| Situación | Comando | Por qué |
|---|---|---|
| Ver qué hay de nuevo en el remoto sin cambiar tus archivos | | |
| Descargar e integrar los cambios del remoto en tu rama actual | | |
| Publicar una rama local en el remoto por primera vez | | |
| Borrar una rama en el servidor remoto | | |
| Comprobar a qué URL apunta `origin` | | |

---

## Solución

!!! tip "Descarga la solución"
    📄 [Solución 0.3 — Tu portafolio en GitHub](plantillas/Actividad_0_3_Solucion.docx){target="_blank" rel="noopener"}

    Esta actividad es de repaso/introducción y no se entrega: practica los pasos por tu cuenta y usa la solución para autoevaluarte al terminar (o si te atascas en algún paso).
