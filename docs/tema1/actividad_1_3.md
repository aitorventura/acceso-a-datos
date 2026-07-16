# 🧪 Actividad 1.3: Acceso a la base de datos con JDBC puro

!!! warning "Descarga la plantilla"
    📄 [Plantilla 1.3 — Acceso a la base de datos con JDBC puro](plantillas/Actividad_1_3_AD_Plantilla.docx){target="_blank" rel="noopener"}

!!! info "Un proyecto nuevo, deliberadamente sin Spring"
    Hoy no tocas GameVault. Vas a crear un proyecto Java suelto, sin Spring Boot ni Hibernate — solo el driver de PostgreSQL — para ver JDBC sin nada que lo oculte. Tienes la teoría de este apartado como referencia (con un ejemplo completo, `Libro`/`Editorial`), pero aquí no hay código dado: el dominio es distinto (`Curso`/`Alumno`) y el código lo escribes tú, adaptando el patrón.

## Qué vas a practicar

- Montar un proyecto Java + Maven desde cero, dentro de su propio Dev Container.
- Levantar tu propia base de datos PostgreSQL con Docker Compose, de nuevo.
- Conectar con `DriverManager` y ejecutar una consulta parametrizada con `PreparedStatement`.
- Recorrer un `ResultSet` mapeando manualmente cada fila.
- Cerrar recursos correctamente con `try-with-resources`.
- Provocar y observar una inyección SQL real, para entender por qué se evita.

---

## Requisitos previos

Docker funcionando y la extensión Dev Containers (las mismas que usaste en la Actividad 1.1). Haber leído la teoría de este apartado antes de empezar — la necesitas para los Pasos 3 y 4, donde no hay código dado.

---

## Paso 0 — Crear el proyecto

Crea una carpeta nueva, separada de tu repositorio de GameVault — por ejemplo, `jdbc-practica`. Dentro, un `pom.xml` mínimo, sin Spring Boot:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.tunombre</groupId>
    <artifactId>jdbc-practica</artifactId>
    <version>1.0.0</version>
    <packaging>jar</packaging>

    <properties>
        <maven.compiler.source>21</maven.compiler.source>
        <maven.compiler.target>21</maven.compiler.target>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.postgresql</groupId>
            <artifactId>postgresql</artifactId>
            <version>42.7.4</version>
        </dependency>
    </dependencies>
</project>
```

Fíjate en lo que **no** hay: ni `spring-boot-starter-parent`, ni `spring-boot-starter-data-jpa`, ni Lombok. Una sola dependencia, el driver JDBC de PostgreSQL — todo lo demás de este proyecto es Java estándar.

---

## Paso 1 — Dev Container y PostgreSQL, otra vez de cero

Mismo patrón que en la Actividad 1.1, un proyecto nuevo con su propio `.devcontainer`. En la raíz de `jdbc-practica`, crea `.devcontainer/docker-compose.yml`:

```yaml
services:
  app:
    image: mcr.microsoft.com/devcontainers/base:bookworm
    volumes:
      - ..:/workspace:cached
    command: sleep infinity

  postgres:
    image: postgres:18-alpine
    environment:
      POSTGRES_DB: jdbc_practica_db
      POSTGRES_USER: jdbc_user
      POSTGRES_PASSWORD: password123
    ports:
      - "5433:5432"
    volumes:
      - postgres_data:/var/lib/postgresql

volumes:
  postgres_data:
```

!!! tip "Puerto `5433`, no `5432`"
    Si tu Dev Container de GameVault está levantado a la vez que este, los dos no pueden publicar el `5432` de tu máquina al mismo tiempo. Aquí lo publicas como `5433:5432` — por fuera del contenedor entras por `5433`, por dentro (donde va a correr tu código Java) sigue siendo el `5432` de siempre.

Y `.devcontainer/devcontainer.json`:

```json
{
  "name": "JDBC Práctica Dev Container",
  "dockerComposeFile": "docker-compose.yml",
  "service": "app",
  "workspaceFolder": "/workspace",
  "forwardPorts": [5433],
  "features": {
    "ghcr.io/devcontainers/features/java:1": {
      "version": "21",
      "installMaven": "true"
    }
  },
  "customizations": {
    "vscode": {
      "extensions": ["vscjava.vscode-java-pack"]
    }
  }
}
```

Abre la carpeta `jdbc-practica` en una ventana nueva de VS Code y elige **"Reopen in Container"**. No hace falta `docker-outside-of-docker` esta vez — este proyecto no va a lanzar contenedores desde dentro de sí mismo, a diferencia de lo que harás en el Tema 3 con Testcontainers en GameVault.

**Captura**: la esquina inferior izquierda de VS Code, con la etiqueta de este Dev Container activo (distinto del de GameVault).

---

## Paso 2 — La tabla y los datos de prueba

Con el contenedor abierto, conéctate a `jdbc_practica_db` y crea el esquema — mismo procedimiento que en la Actividad 1.1: desde una herramienta gráfica en tu equipo (`localhost:5433`, el puerto publicado), o con `docker exec -it <contenedor-postgres> psql -U jdbc_user -d jdbc_practica_db` desde la propia terminal integrada (ahí dentro, el puerto sigue siendo el `5432` interno, no el `5433`):

```sql
CREATE TABLE curso (
    id SERIAL PRIMARY KEY,
    nombre VARCHAR(100) NOT NULL
);

CREATE TABLE alumno (
    id SERIAL PRIMARY KEY,
    nombre VARCHAR(100) NOT NULL,
    email VARCHAR(150) NOT NULL,
    curso_id INTEGER REFERENCES curso(id)
);

INSERT INTO curso (nombre) VALUES ('DAM'), ('DAW');

INSERT INTO alumno (nombre, email, curso_id) VALUES
    ('Ana', 'ana@example.com', 1),
    ('Bruno', 'bruno@example.com', 1),
    ('Carla', 'carla@example.com', 2);
```

`alumno.curso_id` es una clave foránea hacia `curso.id` — el mismo tipo de relación 1:N que ya conoces de `Estudio`/`Videojuego`, esta vez sin ningún ORM de por medio: la tabla la has creado tú, a mano, con SQL puro.

**Comprueba** con `SELECT * FROM alumno;` que las tres filas existen antes de seguir.

---

## Paso 3 — Conectar y consultar, sin código dado

Primero, un `record` `Alumno` — el objeto Java al que vas a mapear cada fila a mano (nada de `@Entity`, esto no lo gestiona Hibernate):

```java
public record Alumno(Long id, String nombre, String email) {}
```

Con eso, crea `src/main/java/com/tunombre/Main.java`, con un método `main` que:

1. Se conecte a `jdbc_practica_db` (URL `jdbc:postgresql://postgres:5432/jdbc_practica_db` — tu código corre **dentro** del contenedor `app`, y `postgres` es el contenedor hermano, no tu propio equipo; el `5433` del Paso 1 es solo el puerto publicado hacia tu máquina para conectarte desde fuera con `psql`/pgAdmin, no el que usa tu código Java desde dentro. Mismo motivo exacto que en la Actividad 1.1).
2. Ejecute esta consulta, parametrizada, para listar los alumnos de un curso concreto:
   ```sql
   SELECT id, nombre, email FROM alumno WHERE curso_id = ?
   ```
3. Recorra el resultado, y por cada fila construya un `Alumno` (con los valores que saques del `ResultSet` con `getXxx`) e imprímalo por consola, con el formato `#id - nombre (email)`.
4. Cierre todos los recursos correctamente.

No hay código dado para este paso — tienes la teoría de este apartado, que cubre exactamente este patrón (abrir conexión, `PreparedStatement` con parámetros, recorrer `ResultSet`, mapear cada fila a un objeto, cerrar con `try-with-resources`) sobre `Libro`/`Editorial`. Adáptalo a `Alumno`/`Curso`: mismo patrón, otro dominio.

**Comprueba**: que la consola imprime los alumnos del curso que hayas puesto, con el formato pedido.

---

## Paso 4 — Por qué el cierre importa

**Pregunta**: si conectaras con un `Connection conn = DriverManager.getConnection(...)` normal, sin `try-with-resources` y sin cerrarla nunca, ¿qué pasaría si ejecutaras tu `main()` 200 veces seguidas? Relaciona tu respuesta con el *pool* de conexiones (HikariCP) que viste en "Conectores y protocolos de acceso a bases de datos" — aunque aquí no haya ningún *pool* de por medio, ¿por qué el problema seguiría siendo real?

---

## Mini-reto — segunda consulta

Repite el mismo patrón del Paso 3 (conexión, `PreparedStatement` parametrizado, recorrido del `ResultSet`, mapeo a un objeto, cierre con `try-with-resources`) para esta consulta nueva:

```sql
SELECT id, nombre FROM curso WHERE nombre = ?
```

Es decir: buscar un curso por su nombre exacto. Necesitas un `record Curso` nuevo (mismo patrón que `Alumno`, con sus propios campos) — el SQL objetivo ya lo tienes, la estructura del código ya la has escrito tú en el Paso 3.

---

## Paso 5 — JDBC puro vs. Spring Data JPA, con la teoría delante

Vuelve a la sección "La misma consulta, dos formas" de la teoría de este apartado — la comparación en pestañas entre `findByEditorialId` (una línea) y el bloque JDBC completo.

**Responde**:

1. Cuenta las líneas de tu propio `main()` del Paso 3. ¿Cuántas te han hecho falta, frente a la única línea que necesitarías con un `AlumnoRepository extends JpaRepository<Alumno, Long>` y un método `findByCursoId`?
2. ¿Qué controlas con JDBC puro que no controlarías igual de fino con JPA (pista: el SQL exacto que se ejecuta, sin que Hibernate decida nada por ti)?
3. ¿Qué se te ha olvidado más fácilmente mientras escribías esto — algo relacionado con abrir/cerrar recursos — que con `JpaRepository` no se te habría podido olvidar?

---

## Paso 6 — Ver la inyección SQL de verdad, no solo leer sobre ella

!!! warning "Esto es una demostración deliberada de un fallo de seguridad"
    Vas a escribir código vulnerable a propósito, en este mismo proyecto suelto, solo para verlo pasar con tus propios ojos. Nunca hagas esto en código que vaya a producción.

Añade un método `buscarPorNombreInseguro(String nombreBuscado)` que construya la consulta **concatenando** el parámetro con `+`, en vez de usar `?` — exactamente el patrón que la teoría marca con el aviso "Nunca concatenes el SQL con datos del usuario". Imprime el SQL resultante por consola antes de ejecutarlo, para poder verlo.

Llámalo primero con un nombre normal (por ejemplo, `"Ana"`) — debería comportarse como esperas. Ahora llámalo con esta entrada exacta:

```
' OR '1'='1
```

**Observa** el SQL impreso por consola antes de ejecutarse, y el resultado que devuelve.

**Preguntas**:

1. ¿Qué SQL se ha ejecutado realmente (cópialo de la consola)? ¿Sigue teniendo la forma "buscar por un nombre concreto"?
2. ¿Cuántas filas ha devuelto, y por qué esa entrada concreta ha conseguido que se devuelvan todas, no solo las que coinciden con un nombre?
3. Vuelve al código del Paso 3: ¿por qué ese mismo ataque, probado contra tu `PreparedStatement` con `?`, no funcionaría?

---

## ✅ Cierre

Has visto, con un proyecto propio y datos reales, la capa que Spring Data JPA te ha estado ocultando desde la Actividad 1.1 — y por qué esa capa existe: no solo por comodidad, también por seguridad (los parámetros de `PreparedStatement`). En la próxima actividad vuelves a GameVault, con procedimientos almacenados invocados desde Java con `JdbcTemplate` — un punto intermedio entre lo que has visto hoy y la comodidad de JPA.
