# Actividad 0.2: Ramas en Git — Trabajo paralelo y fusiones

!!! warning "Descarga la plantilla"
    📄 [Plantilla 0.2 — Ramas en Git: trabajo paralelo y fusiones](plantillas/Actividad_0_2_Plantilla.docx){target="_blank" rel="noopener"}

## Qué vas a practicar

En esta actividad crearás un repositorio desde cero y lo desarrollarás usando ramas. El objetivo es entender cómo Git organiza el trabajo paralelo y qué pasa cuando dos líneas de desarrollo se unen.

Aspectos que cubre la actividad:

- Configurar el alias `git lga` y leer el grafo de ramas.
- Crear ramas con `git branch` y moverte con `git switch`.
- Entender dónde apunta `HEAD` en cada momento.
- Crear una rama y cambiarte en un solo paso con `git switch -c`.
- Hacer una fusión Fast-forward.
- Hacer una fusión 3-way merge cuando las dos ramas han avanzado.
- Resolver un conflicto a mano.
- Borrar ramas con `-d` y `-D`.

---

## Requisitos previos

Configura el alias que usarás durante toda la actividad:

```bash
git config --global alias.lga "log --graph --oneline --all --decorate"
```

Comprueba que Git tiene tu nombre y correo:

```bash
git config --global user.name "Tu Nombre"
git config --global user.email "tuemail@ejemplo.com"
```

---

## Norma de la actividad

!!! warning "Obligatorio en cada paso"
    Antes de ejecutar cualquier comando que cambie el repositorio, escribe **qué esperas que pase** y dónde crees que quedará HEAD. Es lo que se valorará.

---

## Preparación

Crea una carpeta `actividad0_2` fuera de OneDrive / iCloud / Google Drive e inicializa el repositorio:

```bash
git init
git status
```

---

## Parte A — Historial inicial

### Paso 1 — Construir el historial base

Crea `tareas.txt` con tres líneas de contenido (elige tú cuáles). Haz tres commits seguidos:

- Commit 1: el archivo con las tres tareas iniciales.
- Commit 2: añade dos tareas más al final.
- Commit 3: crea `notas.txt` con tres líneas de texto libre y haz commit de ese archivo.

Después de cada commit ejecuta `git lga` y fíjate en dónde se mueve la etiqueta `main` y el puntero `HEAD`.

**Predice antes del tercer commit**: ¿cambiarán de sitio las etiquetas de los commits anteriores?

```bash
git log --oneline
git lga
```

**Preguntas a responder**:

1. ¿Cuántos commits hay en el historial? ¿En qué orden aparecen?
2. ¿Dónde apunta HEAD después del tercer commit?

---

## Parte B — Ramas y HEAD

### Paso 2 — Crear y explorar ramas

**Predice**: si ejecutas `git branch ideas`, ¿dónde quedará HEAD?

```bash
git branch ideas
git lga
git branch
```

**Pregunta**: ¿apuntan `main` e `ideas` al mismo commit o a distintos? ¿Por qué?

Ahora muévete a `ideas`:

**Predice**: después de `git switch ideas`, ¿qué mostrará `HEAD` en `git lga`?

```bash
git switch ideas
git lga
```

**Pregunta**: ¿ha cambiado el historial de commits al cambiar de rama? Explica qué diferencia hay entre `HEAD -> ideas` y `HEAD -> main`.

### Paso 3 — Commit en la rama y volver a main

Sigue en `ideas`. Añade una línea al final de `tareas.txt`:

```
Explorar proyectos de otros compañeros
```

**Predice**: ¿cómo quedará el grafo después de este commit? ¿Se moverá `main`?

```bash
git add tareas.txt
git commit -m "Añade idea de exploración"
git lga
```

Vuelve a `main` y abre `tareas.txt`:

```bash
git switch main
cat tareas.txt
```

**Pregunta**: ¿por qué `tareas.txt` no tiene la línea que acabas de escribir? Explícalo con tus palabras.

---

## Parte C — Fusión Fast-forward

### Paso 4 — Fusionar y limpiar

Estás en `main`. La rama `ideas` tiene un commit que `main` no tiene, pero `main` no ha avanzado.

**Predice**: ¿qué tipo de fusión hará Git? ¿Creará un commit nuevo?

```bash
git merge ideas
git lga
```

**Preguntas a responder**:

1. ¿Qué mensaje ha mostrado Git? ¿Ha creado un commit nuevo o ha movido el puntero?
2. ¿Por qué se llama Fast-forward?

Borra la rama fusionada:

**Predice**: ¿funcionará `-d` o dará error?

```bash
git branch -d ideas
git lga
```

---

## Parte D — Ramas divergentes (3-way merge)

### Paso 5 — Trabajar en paralelo

Crea la rama `mejoras` y muévete en un solo paso:

```bash
git switch -c mejoras
```

**Pregunta**: ¿qué hace `git switch -c` que no hace `git switch` sin `-c`?

Añade una línea al final de `notas.txt` (por ejemplo: `Nota desde la rama mejoras`) y haz commit:

```bash
git add notas.txt
git commit -m "Añade nota desde mejoras"
```

Vuelve a `main` y también haz un commit ahí — añade una línea distinta al final de `tareas.txt`:

```bash
git switch main
# edita tareas.txt → añade: Tarea añadida desde main
git add tareas.txt
git commit -m "Añade tarea desde main"
git lga
```

**Pregunta**: ¿cómo se ve en el grafo que las dos ramas han divergido?

### Paso 6 — 3-way merge

**Predice**: ¿creará Git un commit de fusión? ¿Por qué sí o por qué no?

```bash
git merge mejoras
# Vim: Esc :wq Enter  |  Nano: Ctrl+O Enter Ctrl+X
git lga
```

**Preguntas a responder**:

1. ¿En qué se diferencia visualmente este grafo del del Paso 4?
2. ¿Por qué ha sido necesario un merge commit aquí pero no antes?

---

## Parte E — Conflictos

### Paso 7 — Provocar y resolver un conflicto

Crea la rama `version-a` y cambia la **primera línea** de `tareas.txt` a: `Estudiar con detenimiento los apuntes de Git`

```bash
git switch -c version-a
# edita tareas.txt → cambia la primera línea
git add tareas.txt
git commit -m "Reescribe primera tarea en version-a"
```

Vuelve a `main` y cambia **esa misma primera línea** a algo diferente: `Repasar los apuntes de Git antes del examen`

```bash
git switch main
# edita tareas.txt → cambia la primera línea a algo distinto
git add tareas.txt
git commit -m "Reescribe primera tarea en main"
```

**Predice**: ¿qué va a pasar al intentar fusionar?

```bash
git merge version-a
git status
```

Abre `tareas.txt`. Verás las marcas del conflicto:

```
<<<<<<< HEAD
Repasar los apuntes de Git antes del examen
=======
Estudiar con detenimiento los apuntes de Git
>>>>>>> version-a
```

Decide qué versión conservar (o escribe una que combine ambas), borra las marcas y cierra el merge:

```bash
git add tareas.txt
git commit -m "Resuelve conflicto en tareas.txt"
git lga
```

**Preguntas a responder**:

1. ¿Qué significan las tres marcas del conflicto?
2. ¿Qué versión has conservado y por qué?

---

## Parte F — Borrar ramas: -d vs -D

### Paso 8 — La diferencia entre -d y -D

Crea la rama `experimento`, haz un commit en ella y vuelve a `main` sin fusionarla:

```bash
git switch -c experimento
echo "Experimento descartado" >> notas.txt
git add notas.txt
git commit -m "Cambio experimental"
git switch main
```

**Predice**: ¿funcionará `git branch -d experimento`?

```bash
git branch -d experimento
```

Git da error — fíjate en el mensaje exacto.

**Pregunta**: ¿por qué Git se niega a borrar la rama? ¿Qué pasa con los commits si usas `-D`?

```bash
git branch -D experimento
git lga
```

### Paso 9 — Tabla de decisión

Rellena esta tabla sin ejecutar nada:

| Situación | Comando que usarías | Por qué |
|---|---|---|
| Crear una rama y moverte en un solo paso | | |
| Borrar una rama ya fusionada | | |
| Borrar una rama con trabajo sin fusionar | | |
| Ver el grafo de todas las ramas | | |
| Fusionar cuando main y tu rama han avanzado | | |

---

## Solución

!!! tip "Descarga la solución"
    📄 [Solución 0.2 — Ramas en Git: trabajo paralelo y fusiones](plantillas/Actividad_0_2_Solucion.docx){target="_blank" rel="noopener"}

    Esta actividad es de repaso/introducción y no se entrega: practica los pasos por tu cuenta y usa la solución para autoevaluarte al terminar (o si te atascas en algún paso).
