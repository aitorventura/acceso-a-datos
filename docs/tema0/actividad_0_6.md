# Actividad 0.6: Levantar WordPress con Docker Compose

!!! warning "Descarga la plantilla"
    📄 [Plantilla 0.6 — Levantar WordPress con Docker Compose](plantillas/Actividad_0_6_Plantilla.docx){target="_blank" rel="noopener"}

## Qué vas a practicar

En la teoría has visto Docker Compose con un único servicio. Aquí vas a dar el paso a un caso real: dos contenedores que dependen el uno del otro (una aplicación WordPress y su base de datos), un fallo de conexión real que vas a tener que diagnosticar leyendo logs, y una comprobación de que los datos sobreviven a un reinicio gracias a los volúmenes.

Aspectos que cubre la actividad:

- Levantar varios servicios conectados entre sí con `docker compose up`.
- Diagnosticar un fallo de conexión entre contenedores leyendo `docker compose logs`.
- Entender por qué los servicios se referencian por nombre y no por dirección IP.
- Comprobar que los datos persisten en un volumen tras recrear los contenedores.
- Diferenciar `docker compose down` de `docker compose down -v`.

---

## Requisitos previos

Docker instalado y funcionando. Comprueba que responden sin error:

```bash
docker --version
docker compose version
```

---

## Norma de la actividad

!!! warning "Obligatorio en cada paso marcado como predicción"
    Antes de ejecutar el comando de ese paso, escribe **qué esperas que pase**. Después ejecuta y compara. Esto es lo que se va a valorar: que entiendas por qué pasa lo que pasa, no que copies el resultado.

---

## Preparación

Crea una carpeta `actividad0_6` y, dentro, un fichero `docker-compose.yml` con exactamente este contenido:

```yaml
services:
  mariadb:
    image: mariadb:11
    environment:
      MARIADB_ROOT_PASSWORD: root_pass
      MARIADB_DATABASE: wordpress_db
      MARIADB_USER: TU_USUARIO_BD
      MARIADB_PASSWORD: TU_CONTRASEÑA_BD
    volumes:
      - mariadb_data:/var/lib/mysql

  wordpress:
    image: wordpress:6-apache
    depends_on:
      - mariadb
    environment:
      WORDPRESS_DB_HOST: database
      WORDPRESS_DB_USER: TU_USUARIO_BD
      WORDPRESS_DB_PASSWORD: TU_CONTRASEÑA_BD
      WORDPRESS_DB_NAME: wordpress_db
    ports:
      - "8080:80"
    volumes:
      - wordpress_data:/var/www/html

volumes:
  mariadb_data:
  wordpress_data:
```

Este fichero tiene un error a propósito. No lo corrijas todavía — lo vas a encontrar tú mismo en la Parte B leyendo los logs, igual que pasaría en un proyecto real.

---

## Parte A — Personaliza antes de arrancar

### Paso 1 — Tus propias credenciales de base de datos

Sustituye `TU_USUARIO_BD` y `TU_CONTRASEÑA_BD` por un usuario y contraseña elegidos por ti. **Ojo**: cada uno de los dos valores aparece dos veces en el fichero (una vez en `mariadb` y otra en `wordpress`) — tienen que quedar exactamente iguales en las dos.

**Predicción**: ¿qué crees que pasaría si pusieras una contraseña en `mariadb` y otra distinta en `wordpress`? Escribe tu respuesta antes de continuar.

---

## Parte B — El primer arranque (y el primer fallo)

### Paso 2 — Arrancar

```bash
docker compose up -d
docker compose ps
```

**Predicción**: antes de abrir el navegador, escribe qué esperas ver al entrar en `http://localhost:8080`.

Abre esa dirección en el navegador. Lo más probable es que WordPress muestre un error de conexión a la base de datos.

### Paso 3 — Diagnosticar el fallo

```bash
docker compose logs wordpress
```

**Preguntas a responder**:

1. ¿Qué mensaje de error exacto aparece en el log (o en la propia página web)?
2. Compara el valor de `WORDPRESS_DB_HOST` en tu `docker-compose.yml` con el nombre real que le has dado al servicio de base de datos. ¿Coinciden?
3. ¿Por qué Docker Compose no puede resolver `database` como una dirección válida, si ese nombre no existe en ningún sitio del fichero?

### Paso 4 — Corregir y volver a intentar

Corrige `WORDPRESS_DB_HOST` para que apunte al nombre real del servicio de base de datos que has definido en `services:`.

```bash
docker compose down
docker compose up -d
docker compose logs -f wordpress
```

**Pregunta**: ¿por qué ha hecho falta `docker compose down` antes de volver a `up`, y no bastaba con editar el fichero y esperar? (Pista: piensa en qué momento se leen los valores de `environment` — ¿en cada arranque, o solo cuando el contenedor se crea por primera vez?)

Vuelve a abrir `http://localhost:8080`. Esta vez deberías ver el asistente de instalación de WordPress (elige el idioma y pulsa "Continuar").

---

## Parte C — Comprobar que funciona y que los datos persisten

### Paso 5 — Completar la instalación

Rellena el formulario de instalación de WordPress: título del sitio, usuario administrador y contraseña **elegidos por ti** (no tienen relación con las credenciales de la base de datos del Paso 1 — son dos cosas distintas).

**Captura**: pantalla del panel de administración de WordPress (`/wp-admin`), con tu usuario visible.

### Paso 6 — Crear algo que sobreviva

Crea una entrada (post) nueva cuyo título incluya tu nombre o apellido, para que se identifique como tuya, y publícala.

### Paso 7 — Reiniciar sin perder nada

```bash
docker compose down
docker compose up -d
```

**Predicción**: ¿seguirá existiendo tu entrada después de este reinicio? Razona tu respuesta antes de comprobarlo.

Verifica entrando de nuevo en `http://localhost:8080`.

**Pregunta**: ¿por qué la entrada sigue ahí a pesar de que `docker compose down` elimina los contenedores? Relaciónalo con lo que sabes sobre volúmenes.

### Paso 8 — Borrón y cuenta nueva

```bash
docker compose down -v
```

**Predicción**: ¿qué diferencia habrá esta vez respecto al Paso 7, si volvieras a levantar el entorno?

---

## Preguntas finales

1. Si `wordpress` no tuviera `depends_on: mariadb`, ¿el fallo del Paso 2 sería exactamente el mismo, distinto, o dependería del momento?
2. Si dos compañeros quisierais levantar este mismo `docker-compose.yml` a la vez en el mismo ordenador sin que choquen entre sí, ¿qué tendríais que cambiar?
3. Si tuvieras que hacer una copia de seguridad de todo lo que has creado en WordPress, ¿qué volumen o volúmenes tendrías que conservar, y cuáles no serían imprescindibles?

---

## Solución

!!! tip "Descarga la solución"
    📄 [Solución 0.6 — Levantar WordPress con Docker Compose](plantillas/Actividad_0_6_Solucion.docx){target="_blank" rel="noopener"}

    Esta actividad es de repaso/introducción y no se entrega: practica los pasos por tu cuenta y usa la solución para autoevaluarte al terminar (o si te atascas en algún paso).
