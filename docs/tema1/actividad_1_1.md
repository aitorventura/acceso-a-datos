# 🧪 Actividad 1.1: Docker Compose + PostgreSQL — arranque de las entidades del catálogo

!!! warning "Descarga la plantilla"
    📄 [Plantilla 1.1 — Docker Compose + PostgreSQL: arranque de las entidades del catálogo](plantillas/Actividad_1_1_AD_Plantilla.docx){target="_blank" rel="noopener"}

## Qué es GameVault

**GameVault** es la aplicación que vas a construir, pieza a pieza, durante todo el curso — a la vez desde Acceso a Datos y desde Programación de Servicios y Procesos. Es una API REST que gestiona el catálogo de una tienda de videojuegos: qué juegos hay, qué estudio ha desarrollado cada uno, qué reseñas han dejado los usuarios, y quién puede hacer qué sobre esos datos.

No la construyes de una vez: cada actividad añade una pieza sobre las anteriores, hasta llegar a una aplicación completa con persistencia en dos motores distintos, autenticación y comunicación en tiempo real.

| Pieza | Qué hace | Dónde la construyes |
|---|---|---|
| **Catálogo** | Videojuegos y estudios, con su relación, CRUD completo | Acceso a Datos, Tema 1 |
| **API REST** | Los mismos datos, expuestos por HTTP y documentados con OpenAPI | Programación de Servicios y Procesos, Tema 1 |
| **Seguridad** | Login, roles y rutas protegidas con JWT | Programación de Servicios y Procesos, Tema 2 |
| **Detalles de plataforma** | Información estructurada de cada videojuego (JSONB), con consultas sobre su contenido | Acceso a Datos, Tema 2 |
| **Reseñas** | Opiniones de usuarios sobre cada videojuego, guardadas en MongoDB | Acceso a Datos, Tema 3 |
| **Rendimiento en segundo plano** | Una caché de novedades que se recalienta sola, con hilos, tras cada cambio en el catálogo | Programación de Servicios y Procesos, Tema 3 |
| **Tiempo real** | Un panel de actividad en vivo por WebSocket | Programación de Servicios y Procesos, Tema 4 |

La tabla está en el orden aproximado en que las vas a ir construyendo: Catálogo y API REST arrancan la misma semana, desde los dos módulos a la vez; el resto de piezas se apoya en las anteriores (por ejemplo, las reseñas necesitan que ya exista login y roles).

Hoy construyes la primera pieza: el proyecto vacío, la base de datos, y las dos primeras entidades del catálogo.

!!! info "Práctica guiada"
    Esta es la primera actividad de tu propio GameVault: vas a crear el proyecto desde cero y a dejarlo con una base de datos real y dos entidades JPA funcionando. Sigue los pasos en orden — cada uno se apoya en el anterior.

## Qué vas a practicar

Hasta ahora todo lo que has visto de Spring Boot ha sido teoría y ejemplos de código. Hoy generas **tu propio** proyecto Spring Boot desde cero, lo conviertes en un **Dev Container** que trae PostgreSQL incluido, y creas las entidades `Estudio` y `Videojuego` con su relación, comprobando que Hibernate las traduce en tablas reales.

- Generar un proyecto Spring Boot con Spring Initializr, eligiendo solo las dependencias que necesitas hoy.
- Montar un Dev Container propio para el proyecto, con Java, Maven y PostgreSQL ya listos dentro.
- Definir la estructura de la base de datos con entidades JPA anotadas.
- Verificar que la conexión y el mapeo funcionan de verdad, no solo que "compila".

!!! info "Por qué esta vez todo va dentro de un Dev Container"
    A partir de hoy trabajas con tu propio proyecto, no con ejercicios sueltos — y ese proyecto tiene que arrancar igual en tu portátil, en el ordenador del aula o en el de un compañero. Instalar Java, Maven, Git o el driver de PostgreSQL a mano en cada equipo es justo lo que falla: versiones distintas, permisos de instalación limitados en los equipos del centro, media hora perdida antes de escribir una sola línea. Metiendo todo eso dentro de un Dev Container (lo viste en el Tema 0, Actividad 0.7) te aseguras de que el entorno es siempre el mismo, esté donde esté tu proyecto — y no dependes de qué tenga instalado el equipo en el que te sientes ese día.

---

## Requisitos previos

Docker funcionando y tu editor listo para Dev Containers: Visual Studio Code con la extensión **Dev Containers**, o IntelliJ IDEA Ultimate con su soporte nativo (Actividad 0.7).

---

## Paso 0 — Crear el proyecto desde cero

Ve a [start.spring.io](https://start.spring.io) y genera un proyecto con:

- **Project**: Maven
- **Language**: Java
- **Spring Boot**: la versión estable más reciente de la rama 4.x que te ofrezca el asistente
- **Packaging**: Jar (no War — no vas a desplegar esto en un servidor de aplicaciones externo, el propio `.jar` ya trae el servidor embebido)
- **Java**: 21 — tiene que coincidir con el Java del Dev Container que montas en el Paso 1
- **Group/Artifact**: los que quieras (por ejemplo `com.tunombre` / `gamevault`)
- **Dependencies**: añade solo estas cuatro — ni una más:
    - **Spring Web** (el starter real se llama `spring-boot-starter-webmvc` en esta versión de Spring Boot, no `-web`)
    - **Spring Data JPA**
    - **PostgreSQL Driver**
    - **Lombok**

Pulsa **Generate**, descomprime el `.zip`, y renombra la carpeta a `gamevault` (o el nombre que prefieras) — todavía no la abras en tu IDE.

!!! warning "No añadas todavía el resto de dependencias"
    Tu GameVault terminado va a incluir también MongoDB, RabbitMQ, Redis, seguridad, OpenAPI... pero no las necesitas hoy. Las irás añadiendo actividad a actividad, cuando toque cada una — añadirlas todas de golpe ahora solo te va a confundir sobre qué hace falta para qué.

Fíjate en que Spring Initializr ya te ha creado una clase con `@SpringBootApplication` (el equivalente a `LibreriaApplication` que has visto en la teoría) y la estructura de carpetas `src/main/java` / `src/main/resources` — es exactamente lo que viste "por dentro" en el apartado de teoría, solo que ahora es tuyo y está vacío.

**Pregunta**: ¿por qué crees que Spring Initializr te deja elegir las dependencias en vez de añadirlas todas por defecto?

---

## Paso 1 — El Dev Container: tu entorno y PostgreSQL, juntos

En la teoría del Tema 0 viste que un `devcontainer.json` puede apoyarse en un `docker-compose.yml` en vez de en una sola `image` suelta — así el mismo fichero que describe tu entorno de programación puede levantar, en otro de sus servicios, la base de datos. Es justo lo que vas a montar hoy: un servicio `app` (tu entorno Java) y un servicio `postgres` (el gestor), arrancando los dos a la vez.

En la raíz de tu proyecto, crea la carpeta `.devcontainer` y, dentro, `docker-compose.yml`:

```yaml
services:
  app:
    image: mcr.microsoft.com/devcontainers/base:bookworm
    volumes:
      - ..:/workspace:cached
      - /var/run/docker.sock:/var/run/docker.sock
    command: sleep infinity

  postgres:
    image: postgres:18-alpine
    environment:
      POSTGRES_DB: gamevault_db
      POSTGRES_USER: gamevault_user
      POSTGRES_PASSWORD: password123
    ports:
      - "5432:5432"
    volumes:
      - postgres_data:/var/lib/postgresql

volumes:
  postgres_data:
```

!!! warning "Con Postgres 18, el volumen va en `/var/lib/postgresql`, no en `/var/lib/postgresql/data`"
    Desde la versión 18, la imagen organiza los datos en subcarpetas por versión mayor dentro de `/var/lib/postgresql` — si montas el volumen directamente en `/var/lib/postgresql/data` (la ruta que usaban las versiones anteriores), el contenedor se niega a arrancar y sale con error nada más crearse. Si ya te ha pasado esto, sigue las instrucciones de abajo para limpiar el volumen roto antes de continuar.

`app` parte esta vez de la imagen **base** genérica (la misma familia que ya viste en el Tema 0) — git y las herramientas básicas ya incluidas, pero sin Java todavía. `command: sleep infinity` mantiene el contenedor vivo esperando a que tu editor se conecte (sin este comando, un contenedor sin ningún proceso principal se pararía solo nada más arrancar). `postgres` es exactamente el servicio que ya conoces de la Actividad 0.6, solo que ahora convive con tu entorno de trabajo en el mismo fichero. El segundo `volumes` de `app`, `/var/run/docker.sock:/var/run/docker.sock`, monta dentro del contenedor el mismo Docker que ya tiene tu equipo — no lo vas a necesitar hasta el Tema 3, cuando uses Testcontainers para lanzar contenedores de prueba desde tus propios tests, pero sin este montaje tu contenedor no tendría ningún Docker al que pedírselo.

Y, junto a él, `devcontainer.json`:

```json
{
  "name": "GameVault Dev Container",
  "dockerComposeFile": "docker-compose.yml",
  "service": "app",
  "workspaceFolder": "/workspace",
  "forwardPorts": [8080, 5432],
  "features": {
    "ghcr.io/devcontainers/features/java:1": {
      "version": "21",
      "installMaven": "true"
    },
    "ghcr.io/devcontainers/features/docker-outside-of-docker:1": {}
  },
  "customizations": {
    "vscode": {
      "extensions": ["vscjava.vscode-java-pack", "vmware.vscode-boot-dev-pack"]
    }
  }
}
```

Java ya no viene incluido en la imagen — lo añades como *feature*, igual que `docker-outside-of-docker`: son piezas que se instalan por encima de la imagen base, sin que tengas que elegir una imagen distinta por cada lenguaje o herramienta que necesites. `ghcr.io/devcontainers/features/java:1` instala el JDK, en la versión que le indiques en `"version"`; `"installMaven": "true"` le pide **además** Maven — no lo des por hecho ni lo dejes al valor por defecto de la *feature*: indícalo explícitamente, como aquí, o `mvn` no va a existir dentro del contenedor.

!!! tip "Por qué esta imagen y no una con Java ya incluido"
    Existen imágenes que ya traen el JDK preinstalado (como `mcr.microsoft.com/devcontainers/java`), y en principio ahorran un paso. El problema es que esas imágenes "completas" también traen preinstaladas otras herramientas que no vas a usar (Node.js, Yarn...) — y si el repositorio de alguna de ellas deja de estar disponible o su clave de firma caduca, el contenedor entero puede fallar al construirse por algo que no tiene nada que ver con tu proyecto. Partir de la imagen base y añadir solo las *features* que necesitas (Java, Docker) es más código, pero es más robusto: solo dependes de lo que realmente usas.

!!! tip "La clave `customizations.vscode` es solo para VS Code"
    Si usas IntelliJ IDEA, esas extensiones no hacen falta: el soporte de Java, Maven y Spring Boot ya viene integrado en el propio editor (Ultimate), sin instalar nada aparte.

`service: "app"` le dice a la extensión Dev Containers (o al soporte nativo de IntelliJ) a cuál de los dos servicios del `docker-compose.yml` debe conectar tu editor — al otro, `postgres`, lo levanta igualmente, pero solo como servicio de fondo, sin que tu editor "entre" en él. `forwardPorts` publica hacia tu máquina tanto el `8080` (tu aplicación, cuando la arranques) como el `5432` (PostgreSQL, por si quieres conectarte con una herramienta gráfica desde fuera del contenedor). La *feature* `docker-outside-of-docker` instala dentro de `app` el cliente `docker` (el programa, no un Docker completo) apuntando al socket que acabas de montar — gracias a esto, desde la propia terminal integrada de tu editor vas a poder ejecutar `docker`, `docker compose` o `docker exec` como si estuvieras fuera del contenedor, controlando los mismos contenedores que ve tu sistema operativo.

<div class="tabs-colored" markdown>

=== "🔵 VS Code"
    Abre la carpeta del proyecto en VS Code y, cuando te lo proponga, elige **"Reopen in Container"** (igual que en la Actividad 0.7). La primera vez tarda un poco: está construyendo el contenedor de `app` y levantando `postgres` a la vez.

=== "🟣 IntelliJ IDEA"
    Abre el proyecto en IntelliJ IDEA, abre el fichero `devcontainer.json` y usa el icono **"Create Dev Container"** del margen izquierdo → **"Create Dev Container and Mount Sources…"** (igual que en la Actividad 0.7). Sigue el progreso en la ventana **Services** y pulsa **Connect** cuando termine.

</div>

!!! warning "Si cambias `devcontainer.json`/`docker-compose.yml` después de haber abierto el contenedor"
    Editar estos ficheros no tiene efecto por sí solo — el contenedor ya está construido con la versión anterior. Tienes que reconstruirlo: en VS Code, paleta de comandos (`Ctrl+Shift+P`) → **"Dev Containers: Rebuild Container"**; en IntelliJ IDEA, cierra la conexión desde la ventana **Services** y vuelve a crear el Dev Container desde el icono del margen en `devcontainer.json`. Si algo no aparece como debería (por ejemplo, `mvn` no se encuentra aunque hayas añadido la *feature* de Java), lo primero que hay que comprobar es si de verdad has reconstruido después del último cambio.

**Captura**: el indicativo de Dev Container activo (la esquina inferior izquierda en VS Code; el indicativo de conexión remota en IntelliJ IDEA).

Con el contenedor ya abierto, abre una terminal integrada y comprueba que todo lo necesario está ahí, sin que lo hayas instalado tú:

```bash
java -version
mvn -v
git --version
docker version
```

**Pregunta**: si estos mismos tres comandos los ejecutaras en un ordenador del aula sin Dev Container, ¿qué esperarías que pasara? Relaciónalo con lo que ya viste en la Actividad 0.7 sobre qué corre dentro del contenedor y qué no.

---

## Paso 2 — Configurar la conexión a PostgreSQL

Antes de crear el fichero de perfil, deja también en YAML la configuración común: Spring Initializr te ha generado `src/main/resources/application.properties` con una única línea (`spring.application.name=gamevault`). Bórralo y crea `src/main/resources/application.yml` con el mismo contenido, en formato YAML:

```yaml
spring:
  application:
    name: gamevault
```

Así todo el proyecto queda en YAML desde el principio, sin mezclar formatos entre la configuración común y la de perfil.

Ahora sí, crea `src/main/resources/application-dev.yaml` con la configuración de conexión:

```yaml
spring:
  config:
    activate:
      on-profile: dev
  datasource:
    url: jdbc:postgresql://postgres:5432/gamevault_db
    username: gamevault_user
    password: password123
  jpa:
    hibernate:
      ddl-auto: update
    show-sql: true
```

Fíjate en el host de la URL: `postgres`, no `localhost`. Tu aplicación va a correr **dentro** del contenedor `app`, y `postgres` es un contenedor hermano, no el propio equipo — exactamente el mismo motivo por el que en la Actividad 0.6 WordPress tenía que apuntar al *nombre del servicio* de la base de datos, no a una IP ni a `localhost`. Docker Compose resuelve `postgres` automáticamente a la dirección interna de ese contenedor, porque los dos servicios comparten la misma red creada por el `docker-compose.yml` del Paso 1.

**Predicción**: si dejaras `localhost` en vez de `postgres` en la URL, ¿arrancaría tu aplicación? Escribe tu respuesta y el porqué antes de continuar.

`ddl-auto: update` le dice a Hibernate que, al arrancar, cree o actualice las tablas según tus entidades — es justo lo que vas a comprobar en el Paso 4. `show-sql: true` te va a permitir ver, en la consola, el SQL real que Hibernate ejecuta por debajo — actívalo, te va a servir el resto del curso para entender qué está pasando de verdad.

---

## Paso 3 — Las entidades: `Estudio` y `Videojuego`

### `Estudio`, guiada al completo

Crea el paquete `catalogo` y, dentro, `Estudio.java`:

```java
package com.tunombre.gamevault.catalogo;

import jakarta.persistence.*;
import lombok.Getter;
import lombok.Setter;
import java.util.ArrayList;
import java.util.List;

@Entity
@Table(name = "estudio")
@Getter
@Setter
public class Estudio {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String nombre;
    private String pais;

    @OneToMany(mappedBy = "estudio", cascade = CascadeType.ALL, orphanRemoval = true)
    private List<Videojuego> videojuegos = new ArrayList<>();
}
```

- `@Entity` + `@Table(name = "estudio")`: le dice a Hibernate que esta clase se mapea contra la tabla `estudio` (la crea si no existe, gracias al `ddl-auto: update` del Paso 2).
- `@Id` + `@GeneratedValue(strategy = GenerationType.IDENTITY)`: el identificador, generado por la propia base de datos (una columna autoincremental de PostgreSQL).
- `@OneToMany(mappedBy = "estudio", ...)`: un estudio tiene muchos videojuegos. `mappedBy = "estudio"` dice que el lado que realmente controla la relación en la base de datos (la columna con la clave foránea) está en `Videojuego`, no aquí.
- `cascade = CascadeType.ALL, orphanRemoval = true`: si borras un `Estudio`, se borran también sus videojuegos; y si sacas un videojuego de la lista sin borrar el estudio, ese videojuego "huérfano" también se elimina.

**Pregunta**: ¿por qué crees que tiene sentido que `cascade`/`orphanRemoval` estén en `Estudio → Videojuego`, y no tendría sentido ponerlos al revés (`Videojuego → Estudio`)? Piensa en qué pasaría si borraras un videojuego: ¿debería eso borrar el estudio entero?

### `Videojuego`: el esqueleto guiado, el resto es tuyo

La estructura de `Videojuego` repite un patrón que ya conoces (id autogenerado, campos simples) más una pieza nueva, la relación bidireccional. Te doy el esqueleto con lo nuevo ya resuelto:

```java
package com.tunombre.gamevault.catalogo;

import jakarta.persistence.*;
import lombok.Getter;
import lombok.Setter;
import java.math.BigDecimal;
import java.time.LocalDate;

@Entity
@Table(name = "videojuego")
@Getter
@Setter
public class Videojuego {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    // Mini-reto: añade aquí los campos simples que ya conoces del patrón de Estudio:
    // - titulo (String)
    // - precio (BigDecimal) — usa BigDecimal, no double, para no perder precisión con dinero
    // - fechaLanzamiento (LocalDate)

    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "estudio_id")
    private Estudio estudio;
}
```

`@ManyToOne` es el lado contrario de la relación: muchos videojuegos pertenecen a un estudio. `@JoinColumn(name = "estudio_id")` es la columna real, con la clave foránea, que vivirá en la tabla `videojuego`. `fetch = FetchType.LAZY` significa que Hibernate no carga el `Estudio` completo hasta que de verdad lo pidas (`videojuego.getEstudio()`), en vez de traerlo siempre aunque no lo uses.

**Mini-reto**: completa los tres campos marcados en el comentario, siguiendo exactamente el mismo patrón que usaste en `Estudio` (atributo privado + getter/setter, que ya te genera `@Getter`/`@Setter` de Lombok).

---

## Paso 4 — Verificar que Hibernate ha hecho su trabajo

Desde la terminal integrada de tu editor (la que corre **dentro** del Dev Container), arranca tu aplicación con el perfil `dev` activo:

```bash
./mvnw spring-boot:run -Dspring-boot.run.profiles=dev
```

<div class="tabs-colored" markdown>

=== "🔵 VS Code"
    !!! tip "Alternativa: el botón ▷ Run de VS Code"
        La extensión de Java añade un botón **Run** (o el *CodeLens* `▷ Run` encima del `main`) sobre `GamevaultApplication`. Es más cómodo, pero por defecto arranca la clase directamente con `java`, **sin** el perfil `dev` activo — y sin él, tu aplicación no sabe cómo conectarse a PostgreSQL. Para que el botón Run también use el perfil `dev`, crea `.vscode/launch.json` en tu proyecto:
        ```json
        {
          "version": "0.2.0",
          "configurations": [
            {
              "type": "java",
              "name": "GamevaultApplication (dev)",
              "request": "launch",
              "mainClass": "com.tunombre.gamevault.GamevaultApplication",
              "env": { "SPRING_PROFILES_ACTIVE": "dev" }
            }
          ]
        }
        ```
        Ajusta `mainClass` a tu propio *group id*. A partir de ahora, arrancar con esta configuración desde el panel "Run and Debug" equivale exactamente al comando de arriba.

=== "🟣 IntelliJ IDEA"
    !!! tip "Alternativa: el botón ▷ Run de IntelliJ"
        Al ejecutar por primera vez `GamevaultApplication` desde el *gutter* (▷ junto al `main`), IntelliJ genera una configuración de ejecución (**Run/Debug Configurations**), pero por defecto **sin** el perfil `dev` activo. Edítala (menú desplegable de configuraciones → **Edit Configurations…**) y en el campo **Active profiles** escribe `dev`. No hace falta ningún fichero adicional — a partir de ahora, volver a ejecutar esa misma configuración equivale exactamente al comando de arriba.

</div>

Mira la consola: con `show-sql: true` deberías ver sentencias `create table` (o `alter table`, en arranques posteriores) para `estudio` y `videojuego`.

Ahora conéctate a PostgreSQL para comprobarlo tú mismo. Tienes dos vías, y las dos funcionan igual de bien:

- **Herramienta gráfica desde tu equipo** (pgAdmin, DBeaver): como el Paso 1 ha publicado el puerto `5432` hacia tu máquina, apunta a `localhost:5432` con las credenciales del Paso 1.
- **`psql` desde la propia terminal integrada de tu editor**: gracias al `docker-outside-of-docker` del Paso 1, puedes usar `docker` normalmente aunque esa terminal esté dentro del contenedor `app`. Pero ojo: `docker compose`, a secas, busca un `docker-compose.yml` en la carpeta donde estés (`/workspace`) y no lo encuentra — el tuyo vive en `.devcontainer/`. Indícaselo con `-f`; y como `docker compose` no adivina solo qué contenedores son "los tuyos", indícale también el proyecto con `-p`. No des por hecho el nombre del proyecto (VS Code e IntelliJ no siempre lo nombran igual): averígualo primero con
    ```bash
    docker compose ls
    ```
    y copia el nombre que aparezca en la columna `NAME`. Con ese nombre real (llámalo aquí `<proyecto>`):
    ```bash
    docker compose -f .devcontainer/docker-compose.yml -p <proyecto> ps
    ```
    Localiza ahí el nombre del contenedor de `postgres`, y luego `docker exec -it <ese-nombre> psql -U gamevault_user -d gamevault_db -c "\d videojuego"` — sin salir de tu editor. A partir de aquí, cualquier `docker compose` que ejecutes desde dentro del contenedor va a necesitar esos mismos `-f`/`-p` — con el nombre de proyecto real que acabas de averiguar, no necesariamente `gamevault_devcontainer`.

**Comprueba**:

1. Que existen las tablas `estudio` y `videojuego`.
2. Que `videojuego` tiene una columna `estudio_id` con una clave foránea hacia `estudio`.
3. Que los tipos de columna coinciden con lo esperado (`precio` como `numeric`, `fecha_lanzamiento` como `date`).

**Captura**: el resultado mostrando la clave foránea (de tu herramienta gráfica, o de `\d videojuego` si has usado `psql`).

---

## Pregunta final

Repite la pregunta del Paso 3, pero ahora con la evidencia delante: mirando la tabla `videojuego` que acabas de crear, ¿en qué columna concreta vive la relación con `estudio`? ¿Existe alguna columna equivalente en la tabla `estudio` que apunte a `videojuego`? Relaciona tu respuesta con `mappedBy`.

---

## ✅ Cierre

Al terminar esta actividad tienes: tu propio proyecto Spring Boot (Paso 0), un Dev Container que arranca tu entorno y PostgreSQL a la vez, en cualquier equipo (Paso 1), la conexión configurada (Paso 2) y dos entidades JPA con su relación mapeada y verificada contra la base de datos (Pasos 3-4). Todavía no tienes ningún endpoint HTTP — eso, y el CRUD completo, llega en la Actividad 1.2.
