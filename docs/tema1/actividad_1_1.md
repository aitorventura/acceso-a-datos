# 🧪 Actividad 1.1: Docker Compose + PostgreSQL — arranque de las entidades del catálogo

## Qué es GameVault

**GameVault** es la aplicación que vas a construir, pieza a pieza, durante todo el curso — a la vez desde Acceso a Datos y desde Programación de Servicios y Procesos. Es una API REST que gestiona el catálogo de una tienda de videojuegos: qué juegos hay, qué estudio ha desarrollado cada uno, qué reseñas han dejado los usuarios, y quién puede hacer qué sobre esos datos.

No la construyes de una vez: cada actividad añade una pieza sobre las anteriores, hasta llegar a una aplicación completa con persistencia en dos motores distintos, autenticación y comunicación en tiempo real.

| Pieza | Qué hace | Dónde la construyes |
|---|---|---|
| **Catálogo** | Videojuegos y estudios, con su relación, CRUD completo | Acceso a Datos, Temas 1-2 |
| **API REST** | Los mismos datos, expuestos por HTTP y documentados con OpenAPI | Programación de Servicios y Procesos, Tema 1 |
| **Seguridad** | Login, roles y rutas protegidas con JWT | Programación de Servicios y Procesos, Tema 2 |
| **Detalles de plataforma** | Información estructurada de cada videojuego (JSONB), con consultas sobre su contenido | Acceso a Datos, Tema 3 |
| **Reseñas** | Opiniones de usuarios sobre cada videojuego, guardadas en MongoDB | Acceso a Datos, Tema 4 |
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

Docker funcionando y Visual Studio Code con la extensión **Dev Containers** instalada (Actividad 0.7).

---

## Paso 0 — Crear el proyecto desde cero

Ve a [start.spring.io](https://start.spring.io) y genera un proyecto con:

- **Project**: Maven
- **Language**: Java
- **Spring Boot**: la versión estable más reciente de la rama 4.x que te ofrezca el asistente
- **Group/Artifact**: los que quieras (por ejemplo `com.tunombre` / `gamevault`)
- **Dependencies**: añade solo estas cuatro — ni una más:
    - **Spring Web** (el starter real se llama `spring-boot-starter-webmvc` en esta versión de Spring Boot, no `-web`)
    - **Spring Data JPA**
    - **PostgreSQL Driver**
    - **Lombok**

Pulsa **Generate**, descomprime el `.zip`, y renombra la carpeta a `gamevault` (o el nombre que prefieras) — todavía no la abras en tu IDE.

!!! warning "No añadas todavía el resto de dependencias"
    Tu GameVault terminado va a incluir también MongoDB, RabbitMQ, Redis, seguridad, OpenAPI... pero no las necesitas hoy. Las irás añadiendo actividad a actividad, cuando toque cada una — añadirlas todas de golpe ahora solo te va a confundir sobre qué hace falta para qué.

Fíjate en que Spring Initializr ya te ha creado una clase con `@SpringBootApplication` (el equivalente a `GamevaultApplication.java` que viste la semana pasada) y la estructura de carpetas `src/main/java` / `src/main/resources` — es exactamente lo que viste "por dentro" en el apartado de teoría, solo que ahora es tuyo y está vacío.

**Pregunta**: ¿por qué crees que Spring Initializr te deja elegir las dependencias en vez de añadirlas todas por defecto?

---

## Paso 1 — El Dev Container: tu entorno y PostgreSQL, juntos

En la teoría del Tema 0 viste que un `devcontainer.json` puede apoyarse en un `docker-compose.yml` en vez de en una sola `image` suelta — así el mismo fichero que describe tu entorno de programación puede levantar, en otro de sus servicios, la base de datos. Es justo lo que vas a montar hoy: un servicio `app` (tu entorno Java) y un servicio `postgres` (el gestor), arrancando los dos a la vez.

En la raíz de tu proyecto, crea la carpeta `.devcontainer` y, dentro, `docker-compose.yml`:

```yaml
services:
  app:
    image: mcr.microsoft.com/devcontainers/java:1-21-bookworm
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
      - postgres_data:/var/lib/postgresql/data

volumes:
  postgres_data:
```

`app` usa una imagen que ya trae **Java 21, Maven y Git instalados** — nada que instalar a mano, ni en tu portátil ni en un equipo del aula. `command: sleep infinity` mantiene el contenedor vivo esperando a que tu editor se conecte (sin este comando, un contenedor sin ningún proceso principal se pararía solo nada más arrancar). `postgres` es exactamente el servicio que ya conoces de la Actividad 0.6, solo que ahora convive con tu entorno de trabajo en el mismo fichero. El segundo `volumes` de `app`, `/var/run/docker.sock:/var/run/docker.sock`, monta dentro del contenedor el mismo Docker que ya tiene tu equipo — no lo vas a necesitar hasta el Tema 3, cuando uses Testcontainers para lanzar contenedores de prueba desde tus propios tests, pero sin este montaje tu contenedor no tendría ningún Docker al que pedírselo.

Y, junto a él, `devcontainer.json`:

```json
{
  "name": "GameVault Dev Container",
  "dockerComposeFile": "docker-compose.yml",
  "service": "app",
  "workspaceFolder": "/workspace",
  "forwardPorts": [8080, 5432],
  "features": {
    "ghcr.io/devcontainers/features/docker-outside-of-docker:1": {}
  },
  "customizations": {
    "vscode": {
      "extensions": ["vscjava.vscode-java-pack", "vmware.vscode-boot-dev-pack"]
    }
  }
}
```

`service: "app"` le dice a la extensión Dev Containers a cuál de los dos servicios del `docker-compose.yml` debe conectar tu editor — al otro, `postgres`, lo levanta igualmente, pero solo como servicio de fondo, sin que tu editor "entre" en él. `forwardPorts` publica hacia tu máquina tanto el `8080` (tu aplicación, cuando la arranques) como el `5432` (PostgreSQL, por si quieres conectarte con una herramienta gráfica desde fuera del contenedor). La *feature* `docker-outside-of-docker` instala dentro de `app` el cliente `docker` (el programa, no un Docker completo) apuntando al socket que acabas de montar — gracias a esto, desde la propia terminal integrada de VS Code vas a poder ejecutar `docker`, `docker compose` o `docker exec` como si estuvieras fuera del contenedor, controlando los mismos contenedores que ve tu sistema operativo.

Ahora abre la carpeta del proyecto en VS Code y, cuando te lo proponga, elige **"Reopen in Container"** (igual que en la Actividad 0.7). La primera vez tarda un poco: está construyendo el contenedor de `app` y levantando `postgres` a la vez.

**Captura**: la esquina inferior izquierda de VS Code, con la etiqueta del Dev Container activo.

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

Crea `src/main/resources/application-dev.yaml` con la configuración de conexión:

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

!!! tip "El JSONB de `detallesPlataforma` no es para hoy"
    La entidad `Videojuego` completa de GameVault incluye también un campo `detallesPlataforma` con anotaciones de JSONB. Omítelo por completo en tu proyecto por ahora — se trabaja en el Tema 3, cuando tengas la base necesaria para entenderlo.

---

## Paso 4 — Verificar que Hibernate ha hecho su trabajo

Desde la terminal integrada de VS Code (la que corre **dentro** del Dev Container), arranca tu aplicación con el perfil `dev` activo:

```bash
./mvnw spring-boot:run -Dspring-boot.run.profiles=dev
```

Mira la consola: con `show-sql: true` deberías ver sentencias `create table` (o `alter table`, en arranques posteriores) para `estudio` y `videojuego`.

Ahora conéctate a PostgreSQL para comprobarlo tú mismo. Tienes dos vías, y las dos funcionan igual de bien:

- **Herramienta gráfica desde tu equipo** (pgAdmin, DBeaver): como el Paso 1 ha publicado el puerto `5432` hacia tu máquina, apunta a `localhost:5432` con las credenciales del Paso 1.
- **`psql` desde la propia terminal integrada de VS Code**: gracias al `docker-outside-of-docker` del Paso 1, puedes usar `docker` normalmente aunque esa terminal esté dentro del contenedor `app`. Ejecuta `docker compose ps` para localizar el nombre del contenedor de `postgres`, y luego `docker exec -it <ese-nombre> psql -U gamevault_user -d gamevault_db -c "\d videojuego"` — sin salir de VS Code.

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
