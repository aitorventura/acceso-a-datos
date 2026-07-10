# 🧪 Actividad 1.1: Docker Compose + PostgreSQL — arranque de las entidades del catálogo

!!! info "Práctica guiada"
    Esta es la primera actividad de tu propio GameVault: vas a crear el proyecto desde cero y a dejarlo con una base de datos real y dos entidades JPA funcionando. Sigue los pasos en orden — cada uno se apoya en el anterior.

## Qué vas a practicar

Hasta ahora todo lo que has visto de Spring Boot ha sido teoría y ejemplos de código. Hoy generas **tu propio** proyecto Spring Boot desde cero, levantas un PostgreSQL con Docker Compose (como hiciste con MariaDB en la Actividad 0.6, pero ahora para tu propia aplicación) y creas las entidades `Estudio` y `Videojuego` con su relación, comprobando que Hibernate las traduce en tablas reales.

- Generar un proyecto Spring Boot con Spring Initializr, eligiendo solo las dependencias que necesitas hoy.
- Levantar un gestor de base de datos independiente (PostgreSQL) con Docker Compose.
- Definir la estructura de la base de datos con entidades JPA anotadas.
- Verificar que la conexión y el mapeo funcionan de verdad, no solo que "compila".

---

## Requisitos previos

Docker funcionando (Actividad 0.6) y un IDE con soporte de Java/Maven (IntelliJ, VS Code con extensión Java, o tu Dev Container del Tema 0).

---

## Paso 0 — Crear el proyecto desde cero

Ve a [start.spring.io](https://start.spring.io) (o al asistente equivalente de tu IDE: *File → New → Spring Starter Project* en la mayoría) y genera un proyecto con:

- **Project**: Maven
- **Language**: Java
- **Spring Boot**: la versión estable más reciente de la rama 4.x que te ofrezca el asistente
- **Group/Artifact**: los que quieras (por ejemplo `com.tunombre` / `gamevault`)
- **Dependencies**: añade solo estas cuatro — ni una más:
    - **Spring Web** (el starter real se llama `spring-boot-starter-webmvc` en esta versión de Spring Boot, no `-web`)
    - **Spring Data JPA**
    - **PostgreSQL Driver**
    - **Lombok**

Pulsa **Generate**, descomprime el `.zip` e impórtalo en tu IDE como proyecto Maven.

!!! warning "No añadas todavía el resto de dependencias"
    La referencia tiene también MongoDB, RabbitMQ, Redis, seguridad, OpenAPI... No las necesitas hoy. Las irás añadiendo actividad a actividad, cuando toque cada una — añadirlas todas de golpe ahora solo te va a confundir sobre qué hace falta para qué.

Abre el `pom.xml` generado y localiza las cuatro dependencias que acabas de elegir. Fíjate en que Spring Initializr ya te ha creado también una clase con `@SpringBootApplication` (el equivalente a `GamevaultApplication.java` que viste la semana pasada) y la estructura de carpetas `src/main/java` / `src/main/resources` — es exactamente lo que viste "por dentro" en el apartado de teoría, solo que ahora es tuyo y está vacío.

**Pregunta**: ¿por qué crees que Spring Initializr te deja elegir las dependencias en vez de añadirlas todas por defecto?

---

## Paso 1 — PostgreSQL con Docker Compose

En la raíz de tu proyecto, crea un `docker-compose.yml`:

```yaml
services:
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

Línea a línea, ya lo conoces de la Actividad 0.6: `image` fija la versión exacta del gestor (`postgres:18-alpine`); `environment` crea la base de datos y el usuario la primera vez que arranca el contenedor; `ports` publica el `5432` de tu máquina hacia el `5432` del contenedor; `volumes` guarda los datos fuera del contenedor para que sobrevivan a un reinicio.

Levántalo:

```bash
docker compose up -d
docker compose ps
```

Ahora crea `src/main/resources/application-dev.yaml` con la configuración de conexión:

```yaml
spring:
  config:
    activate:
      on-profile: dev
  datasource:
    url: jdbc:postgresql://localhost:5432/gamevault_db
    username: gamevault_user
    password: password123
  jpa:
    hibernate:
      ddl-auto: update
    show-sql: true
```

`spring.datasource.*` es cómo tu aplicación encuentra y se autentica contra el PostgreSQL que acabas de levantar. `ddl-auto: update` le dice a Hibernate que, al arrancar, cree o actualice las tablas según tus entidades — es justo lo que vas a comprobar en el Paso 3. `show-sql: true` te va a permitir ver, en la consola, el SQL real que Hibernate ejecuta por debajo — actívalo, te va a servir el resto del curso para entender qué está pasando de verdad.

**Predicción**: si ahora mismo arrancaras tu aplicación, ¿aparecería alguna tabla en la base de datos? Escribe tu respuesta y el porqué antes de continuar.

---

## Paso 2 — Las entidades: `Estudio` y `Videojuego`

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

- `@Entity` + `@Table(name = "estudio")`: le dice a Hibernate que esta clase se mapea contra la tabla `estudio` (la crea si no existe, gracias al `ddl-auto: update` del Paso 1).
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

## Paso 3 — Verificar que Hibernate ha hecho su trabajo

Arranca tu aplicación con el perfil `dev` activo (en IntelliJ: *Edit Configurations* → *Active profiles* = `dev`; por línea de comandos: `./mvnw spring-boot:run -Dspring-boot.run.profiles=dev`).

Mira la consola: con `show-sql: true` deberías ver sentencias `create table` (o `alter table`, en arranques posteriores) para `estudio` y `videojuego`.

Conéctate con la herramienta que prefieras (pgAdmin, DBeaver, o `psql` desde la terminal):

```bash
docker exec -it <nombre-del-contenedor-postgres> psql -U gamevault_user -d gamevault_db -c "\d videojuego"
```

**Comprueba**:

1. Que existen las tablas `estudio` y `videojuego`.
2. Que `videojuego` tiene una columna `estudio_id` con una clave foránea hacia `estudio`.
3. Que los tipos de columna coinciden con lo esperado (`precio` como `numeric`, `fecha_lanzamiento` como `date`).

**Captura**: el resultado de `\d videojuego` (o el equivalente gráfico de tu herramienta) mostrando la clave foránea.

---

## Pregunta final

Repite la pregunta del Paso 2, pero ahora con la evidencia delante: mirando la tabla `videojuego` que acabas de crear, ¿en qué columna concreta vive la relación con `estudio`? ¿Existe alguna columna equivalente en la tabla `estudio` que apunte a `videojuego`? Relaciona tu respuesta con `mappedBy`.

---

## ✅ Cierre

Al terminar esta actividad tienes: tu propio proyecto Spring Boot (Paso 0), un PostgreSQL real levantado con Docker Compose (Paso 1) y dos entidades JPA con su relación mapeada y verificada contra la base de datos (Pasos 2-3). Todavía no tienes ningún endpoint HTTP — eso, y el CRUD completo, llega en la Actividad 1.2.
