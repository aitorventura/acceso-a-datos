# Actividad 0.7: Tu primer Dev Container

!!! warning "Descarga la plantilla"
    📄 [Plantilla 0.7 — Tu primer Dev Container](plantillas/Actividad_0_7_Plantilla.docx){target="_blank" rel="noopener"}

## Qué vas a practicar

En la teoría has leído que, al reabrir un proyecto "in container", tu editor deja de trabajar sobre tu propio sistema operativo y pasa a trabajar dentro del contenedor — aunque la ventana se vea exactamente igual. Aquí lo vas a comprobar en primera persona, comparando con tus propios ojos qué responde tu sistema operativo y qué responde el contenedor a las mismas preguntas.

---

## Requisitos previos

- Docker instalado y funcionando.
- Visual Studio Code con la extensión **Dev Containers** instalada.

---

## Paso 1 — Crear el proyecto

Crea una carpeta `actividad0_7` y, dentro, una subcarpeta `.devcontainer` con un fichero `devcontainer.json`:

```json
{
  "name": "Mi primer Dev Container",
  "image": "mcr.microsoft.com/devcontainers/base:debian-12"
}
```

---

## Paso 2 — Abrir en el contenedor

Abre la carpeta `actividad0_7` en VS Code. Debería aparecer, abajo a la derecha, una notificación ofreciendo **"Reopen in Container"**. Si no aparece, ábrela tú mismo desde la paleta de comandos (`Ctrl+Shift+P` / `Cmd+Shift+P`) buscando **"Dev Containers: Reopen in Container"**.

**Captura**: la notificación o el proceso de construcción del contenedor mientras arranca.

Espera a que termine. La primera vez tarda un poco más porque tiene que descargar la imagen.

---

## Paso 3 — Comprobar que estás dentro

Mira la esquina inferior izquierda de la ventana de VS Code.

**Captura**: esa esquina, donde debería verse una etiqueta con el nombre del Dev Container activo.

**Pregunta**: ¿qué texto muestra exactamente esa etiqueta?

---

## Paso 4 — Comparar terminal de dentro y de fuera

Con el contenedor activo, abre una terminal integrada en VS Code (`Ctrl+ñ` o desde el menú *Terminal*) y ejecuta:

```bash
cat /etc/os-release
whoami
pwd
```

**Captura**: la salida de estos tres comandos.

Ahora, **sin cerrar VS Code**, abre una terminal de tu propio sistema operativo (la que usarías normalmente, fuera del editor) y ejecuta los mismos tres comandos.

**Captura**: la salida de esa segunda terminal.

**Preguntas a responder**:

1. ¿Qué diferencias hay entre las dos salidas de `cat /etc/os-release` (o su equivalente si tu sistema operativo no tiene ese fichero)?
2. ¿El resultado de `pwd` es el mismo en las dos terminales? ¿Tiene sentido que lo sea o que no lo sea?
3. Si las dos terminales están abiertas "dentro de VS Code, en la misma ventana", ¿por qué devuelven cosas distintas?

---

## Paso 5 — Volver a tu entorno local

Desde la paleta de comandos, busca **"Dev Containers: Reopen Folder Locally"**.

**Captura**: la esquina inferior izquierda después de volver, mostrando que la etiqueta del Dev Container ha desaparecido.

---

## Pregunta final

Si borraras el contenedor (`docker rm`) y volvieras a hacer "Reopen in Container" sobre la misma carpeta, ¿qué esperas que pase con los archivos que veías dentro de `/workspace`? Razona tu respuesta pensando en qué se monta desde tu ordenador y qué se reconstruye desde la imagen.

---

## Solución

!!! tip "Descarga la solución"
    📄 [Solución 0.7 — Tu primer Dev Container](plantillas/Actividad_0_7_Solucion.docx){target="_blank" rel="noopener"}

    Esta actividad es de repaso/introducción y no se entrega: practica los pasos por tu cuenta y usa la solución para autoevaluarte al terminar (o si te atascas en algún paso).
