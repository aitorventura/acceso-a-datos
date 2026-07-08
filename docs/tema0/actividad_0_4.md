# Actividad 0.4: Contribuir al portafolio de un compañero

!!! warning "Descarga la plantilla"
    📄 [Plantilla 0.4 — Contribuir al portafolio de un compañero](plantillas/Actividad_0_4_Plantilla.docx){target="_blank" rel="noopener"}

!!! warning "Antes de empezar"
    Lee la actividad entera antes de ejecutar nada.

---

## ¿Qué vas a practicar?

En los proyectos reales nadie te da permisos de escritura en el repositorio principal desde el primer día. La forma habitual de contribuir es: hacer un fork, trabajar en una rama propia y proponer los cambios mediante Pull Request. El mantenedor los revisa y decide si los acepta, los rechaza o pide modificaciones.

En esta actividad trabajaréis en parejas. Cada miembro tendrá dos roles: **contribuyente** (hace el fork y la PR hacia el portafolio del compañero) y **mantenedor** (revisa y responde la PR que le llega en su propio portafolio). Ambos roles se ejercen a la vez: mientras abres una PR en el repo de tu compañero, él está abriendo una en el tuyo.

## Requisitos previos

- Repositorio `mi_portfolio` de la actividad 0.3 publicado en GitHub y con contenido en el `README.md`.
- El repositorio debe ser **público** para que tu compañero pueda hacer el fork.
- Las **Issues** deben estar activadas en tu repositorio: en GitHub, entra a **Settings → General → Features** y marca la casilla **Issues**. Sin esto, tu compañero no podrá abrir el Issue de la Parte D en tu `mi_portfolio`.
- Compañero asignado por el profesor.

---

## Parte A — El Fork

**Como contribuyente**, vas a trabajar sobre el `mi_portfolio` de tu compañero.

**Paso 1.** Abre el repositorio `mi_portfolio` de tu compañero en GitHub.

!!! warning "Predicción antes de actuar"
    ¿Apareces tú como colaborador de su repositorio? ¿Podrías hacer `git push` directamente si clonaras su repo sin fork? Anota tu respuesta antes de comprobarlo.

Intenta entrar a **Settings** de su repositorio. ¿Ha coincidido tu predicción con lo que has visto?

**Paso 2.** Haz un fork del repositorio: pulsa el botón **Fork** en la esquina superior derecha. GitHub crea una copia bajo tu usuario.

Clona tu fork en local:

```bash
git clone <URL_DE_TU_FORK>
cd mi_portfolio
```

**Paso 3.** Crea una rama con un nombre que describa tu contribución:

```bash
git switch -c mejora/seccion-herramientas
```

Añade algo real al `README.md` de tu compañero. Elige una opción:

- Una sección `## Herramientas que uso` con 3-5 tecnologías que conozcas.
- Corrección de erratas o mejora del texto existente.
- Una sección `## En qué estoy trabajando` con un proyecto actual o futuro.

**Paso 4.** Confirma y sube los cambios:

```bash
git add README.md
git commit -m "Añade sección de herramientas al portafolio"
git push -u origin mejora/seccion-herramientas
```

---

## Parte B — Abrir la Pull Request

**Paso 5.** Abre la Pull Request desde tu fork hacia el repositorio de tu compañero.

!!! warning "Predicción antes de actuar"
    Antes de abrir el formulario de PR, anota qué valor crees que tendrá cada uno de los cuatro selectores: **Base repository**, **Base**, **Head repository** y **Compare**.

Abre el formulario y comprueba los selectores. Asegúrate de que la PR apunta al repositorio de tu compañero (no al tuyo) y que la rama base es `main`.

Escribe:

- Un **título** que resuma el cambio en una frase corta.
- Una **descripción** que explique qué has añadido y por qué crees que aporta valor.

**Pregunta B.1.** ¿Ha coincidido tu predicción sobre los selectores? ¿Cuál te ha sorprendido y por qué?

---

## Parte C — Revisar la PR que has recibido

**Como mantenedor**, tu compañero ha abierto una PR en tu `mi_portfolio`. Ahora te toca decidir.

**Paso 6.** Entra a tu repositorio en GitHub y abre la pestaña **Pull requests**. Revisa la PR con criterio:

- ¿El cambio propuesto tiene sentido para tu portafolio?
- ¿El mensaje de commit es descriptivo?
- ¿El texto está bien escrito y no rompe nada de lo que ya tenías?

Tienes dos opciones:

- Si lo apruebas: deja un comentario breve explicando por qué lo aceptas y haz el merge.
- Si pides cambios: usa el botón **Request changes** y explica exactamente qué debe modificar tu compañero.

**Pregunta C.1.** ¿Qué criterios has usado para decidir si aceptar o pedir cambios? Justifica tu decisión en 3-5 frases.

---

## Parte D — Issues: primero el ticket, luego el código

En proyectos con varios colaboradores, la práctica habitual es abrir un Issue antes de ponerse a trabajar. Así el equipo sabe qué vas a hacer y se evita trabajo duplicado.

**Paso 7.** Abre un Issue en el repositorio de tu compañero describiendo una segunda mejora distinta a la que ya has hecho. Por ejemplo: *"Añadir sección de contacto con enlace a LinkedIn"*.

**Paso 8.** En tu fork, crea una nueva rama, implementa esa mejora y abre una segunda PR. En la descripción escribe:

```
Closes #N
```

donde `N` es el número del Issue que acabas de abrir.

!!! warning "Predicción antes de actuar"
    ¿Qué crees que ocurrirá con el Issue cuando tu compañero acepte y fusione esta PR? Anota tu predicción antes de que lo haga.

Cuando tu compañero haga el merge, comprueba el estado del Issue. ¿Ha coincidido tu predicción?

**Pregunta D.1.** ¿Cuál es la ventaja de enlazar una PR con un Issue con `Closes #N` frente a cerrar el Issue manualmente después del merge?

---

## Solución

!!! tip "Descarga la solución"
    📄 [Solución 0.4 — Contribuir al portafolio de un compañero](plantillas/Actividad_0_4_Solucion.docx){target="_blank" rel="noopener"}

    Esta actividad es de repaso/introducción y no se entrega: practica los pasos por tu cuenta y usa la solución para autoevaluarte al terminar (o si te atascas en algún paso).
