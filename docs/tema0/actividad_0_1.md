# Actividad 0.1: Git local — construir historial y saber volver atrás

!!! warning "Descarga la plantilla"
    📄 [Plantilla 0.1 — Git local: construir historial y saber volver atrás](plantillas/Actividad_0_1_Plantilla.docx){target="_blank" rel="noopener"}

## Qué vas a practicar

En esta actividad crearás un repositorio desde cero, construirás un historial de cambios y practicarás los comandos para volver atrás cuando algo sale mal. No se trata solo de ejecutar comandos: en cada paso tendrás que **predecir** qué va a pasar antes de hacerlo y **razonar** qué opción elegirías en distintas situaciones.

Aspectos que cubre la actividad:

- Inicializar un repositorio y configurar Git.
- Añadir archivos y hacer commits con mensajes claros.
- Leer el historial con `git log` y examinar commits concretos con `git show`.
- Descartar cambios antes de hacer commit con `git restore`.
- Sacar archivos del staging sin perder los cambios.
- Recuperar una versión anterior de un archivo con `git restore --source`.
- Deshacer un commit de forma segura con `git revert`.
- Entender la diferencia entre `git reset --soft`, `--mixed` y `--hard`.
- Ignorar archivos con `.gitignore`.

---

## Requisitos previos

Antes de empezar, comprueba que Git está instalado y configurado:

```bash
git --version
git config --global user.name "Tu Nombre"
git config --global user.email "tuemail@ejemplo.com"
```

Si `user.name` y `user.email` ya están configurados de sesiones anteriores, no hace falta repetirlo.

---

## Norma de la actividad

!!! warning "Obligatorio en cada paso"
    Antes de ejecutar cualquier comando que modifique el repositorio (add, commit, restore, revert, reset…), escribe **qué esperas que pase**. Después ejecuta y comprueba si has acertado.

    Esto es lo que se va a valorar: que entiendas lo que hace cada comando, no que lo copies sin más.

---

## Preparación del repositorio

Crea una carpeta llamada `actividad0_1` en un lugar de tu disco que **no esté dentro de OneDrive, iCloud ni Google Drive** (esos sistemas de sincronización y Git no se llevan bien).

Inicializa el repositorio dentro de esa carpeta:

```bash
git init
git status
```

---

## Parte A — Construir el historial

### Paso 1 — Primer commit

Crea un archivo llamado `recetas.txt` con exactamente este contenido (puedes cambiar los nombres de platos, pero tiene que haber tres):

```
Tortilla de patatas
Gazpacho
Paella valenciana
```

Antes de añadir nada: **¿qué estado tiene el archivo según `git status`?** Escríbelo.

Después:

```bash
git add recetas.txt
git status
git commit -m "Añade recetas iniciales"
```

### Paso 2 — Segundo commit

Añade dos recetas más al final de `recetas.txt`. Elige las que quieras.

**Predice**: ¿qué mostrará `git status`? ¿Y `git diff`? Escríbelo antes de ejecutar.

```bash
git status
git diff
git add recetas.txt
git commit -m "Añade dos recetas nuevas"
```

### Paso 3 — Tercer commit con dos archivos

- Crea un archivo `ingredientes.txt` con los ingredientes de una de tus recetas (al menos 5 líneas).
- Crea también un archivo `borrador.txt` con el texto `trabajo en progreso`.

Haz un commit que **solo incluya** `ingredientes.txt`. El archivo `borrador.txt` no debe estar en este commit.

```bash
git add ingredientes.txt
git commit -m "Añade ingredientes de una receta"
git status
```

**Pregunta**: ¿por qué `borrador.txt` sigue apareciendo como untracked? ¿Qué tendría que pasar para que desapareciera del todo?

### Paso 4 — Cuarto commit con un error deliberado

Edita `recetas.txt` y **elimina** una de las recetas que añadiste en el Paso 1 (una de las tres originales). Luego haz commit:

```bash
git add recetas.txt
git commit -m "Elimina receta por error"
```

Ahora tienes un historial con cuatro commits. Consúltalo:

```bash
git log --oneline
```

Fíjate en el resultado: lo necesitarás para el resto de pasos.

---

## Parte B — Leer el historial

### Paso 5 — Examinar un commit concreto

Usa `git show` con el hash del **segundo commit** (el que añadió dos recetas nuevas):

```bash
git show <hash>
```

**Preguntas a responder**:

1. ¿Qué líneas aparecen en verde (con `+`)? ¿Qué representan?
2. ¿Qué líneas aparecen en rojo (con `-`)? ¿Cuántas hay en este commit?
3. ¿Para qué sirve `git show` en tu día a día como desarrollador?

---

## Parte C — Descartar cambios antes de commit

### Paso 6 — Descartar cambios en el área de trabajo

Modifica `ingredientes.txt`: borra todas las líneas y escribe `borrador`. No hagas commit.

**Predice**: ¿qué mostrará `git status`? Escríbelo.

Ahora descarta ese cambio sin borrar el archivo ni usar Ctrl+Z:

```bash
git restore ingredientes.txt
cat ingredientes.txt
```

**Pregunta**: ¿qué ha pasado? ¿El archivo ha recuperado el contenido que tenía en el último commit?

### Paso 7 — Sacar un archivo del staging

Añade `ingredientes.txt` al staging pero **no hagas commit**:

```bash
echo "Aceite de oliva extra virgen" >> ingredientes.txt
git add ingredientes.txt
git status
```

**Predice**: ¿qué verás en `git status` en este momento?

Ahora quieres sacar `ingredientes.txt` del staging pero **sin perder el cambio** (la línea que has añadido debe seguir en el archivo):

```bash
git restore --staged ingredientes.txt
git status
cat ingredientes.txt
```

**Preguntas a responder**:

1. ¿El archivo tiene todavía la línea nueva o ha desaparecido?
2. ¿En qué se diferencia `git restore --staged` de `git restore` (sin `--staged`)?

---

## Parte D — Recuperar una versión anterior de un archivo

### Paso 8 — Recuperar el archivo desde un commit pasado

En el Paso 4 borraste una receta por error. Ahora quieres recuperar `recetas.txt` tal como estaba en el **segundo commit** (antes del error).

Busca el hash del segundo commit con `git log --oneline` y ejecuta:

```bash
git restore --source=<hash-segundo-commit> recetas.txt
cat recetas.txt
git status
```

**Preguntas a responder**:

1. ¿Cuántas recetas tiene el archivo ahora?
2. ¿`git status` dice que el archivo está modified o staged? ¿Por qué?
3. ¿Se ha creado un nuevo commit automáticamente? ¿Qué tendrías que hacer para guardar este estado?

Haz commit de esta recuperación:

```bash
git add recetas.txt
git commit -m "Recupera recetas del segundo commit"
git log --oneline
```

---

## Parte E — Deshacer commits

### Paso 9 — git revert: deshacer sin borrar historial

Antes de practicar `git revert` necesitas el directorio limpio (sin cambios pendientes). En el Paso 7 dejaste una línea nueva en `ingredientes.txt` sin commitear. Confírmala ahora:

```bash
git status
git add ingredientes.txt
git commit -m "Añade ingrediente nuevo"
git status
```

Ahora el directorio está limpio. Crea un error deliberado para practicar el revert:

```bash
echo "ESTA LÍNEA ES UN ERROR" >> recetas.txt
git add recetas.txt
git commit -m "Añade línea incorrecta por error"
git log --oneline
```

**Predice**: ¿qué hará `git revert HEAD`? ¿Borrará ese commit o creará uno nuevo encima?

Ahora deshaz ese commit **sin borrar el historial** (simula que el proyecto ya lo has subido a un servidor y otra persona podría haberlo descargado):

```bash
git revert HEAD
git log --oneline
git show HEAD
```

!!! tip "El editor de mensajes"
    Al ejecutar `git revert`, Git abrirá un editor para que confirmes el mensaje del commit de revert. Si se abre Vim, escribe `:wq` y pulsa Enter para guardar y salir. Si prefieres evitarlo, puedes usar `git revert HEAD --no-edit`.

**Preguntas a responder**:

1. ¿Cuántos commits hay ahora en el historial?
2. ¿`git revert` ha borrado el commit del error o ha creado uno nuevo encima?
3. ¿En qué situaciones usarías `git revert` en lugar de `git reset`?

### Paso 10 — git reset --soft

Haz un commit de prueba que puedas deshacer. Añade una línea a `ingredientes.txt` y haz commit:

```bash
echo "Sal al gusto" >> ingredientes.txt
git add ingredientes.txt
git commit -m "Commit de prueba para reset"
```

Ahora deshaz ese commit con `--soft`:

```bash
git reset --soft HEAD~1
git status
git log --oneline
```

**Predice antes de ejecutar**: ¿dónde van los cambios del commit deshecho — al staging, al área de trabajo o desaparecen?

**Preguntas a responder**:

1. ¿El commit ha desaparecido del log?
2. ¿Los cambios están staged o unstaged?
3. ¿Qué tendrías que hacer ahora para rehacer el commit con un mensaje diferente?

### Paso 11 — git reset --mixed

Haz otro commit de prueba:

```bash
git add ingredientes.txt
git commit -m "Otro commit de prueba"
```

Deshaz con `--mixed`:

```bash
git reset --mixed HEAD~1
git status
```

**Preguntas a responder**:

1. ¿En qué se diferencia el resultado de `--soft` respecto a `--mixed`?
2. ¿Los cambios siguen en el archivo o han desaparecido?
3. ¿Cuándo elegirías `--mixed` en lugar de `--soft`?

### Paso 12 — Tabla de decisión

Rellena esta tabla sin ejecutar nada — razona a partir de lo que has practicado:

| Situación | Comando que usarías | Por qué |
|---|---|---|
| Hiciste commit con un mensaje mal escrito y quieres corregirlo | | |
| Quieres deshacer los últimos cambios del archivo sin perder el commit | | |
| Borraste accidentalmente un párrafo y todavía no has hecho commit | | |
| Alguien más ya ha descargado tu commit y quieres deshacerlo de forma segura | | |
| Hiciste commit pero olvidaste añadir un archivo | | |

---

## Parte F — Ignorar archivos

### Paso 13 — .gitignore

Crea archivos que no deberían estar en el repositorio:

```bash
echo "log de prueba" > 20250101.log
echo "otro log" > 20250628.log
```

Configura Git para ignorar todos los archivos `.log` y el archivo `borrador.txt`:

1. Crea el archivo `.gitignore`.
2. Añade las reglas necesarias.
3. Comprueba con `git status` que los archivos ignorados ya no aparecen como untracked.
4. Haz commit: `"Configura .gitignore"`.

**Pregunta**: ¿qué pasa si intentas hacer `git add 20250101.log` después de haberlo añadido al `.gitignore`?

---

## Solución

!!! tip "Descarga la solución"
    📄 [Solución 0.1 — Git local: construir historial y saber volver atrás](plantillas/Actividad_0_1_Solucion.docx){target="_blank" rel="noopener"}

    Esta actividad es de repaso/introducción y no se entrega: practica los pasos por tu cuenta y usa la solución para autoevaluarte al terminar (o si te atascas en algún paso).
