<a id="conectores-y-protocolos"></a>

# 🧩 2. Conectores y protocolos de acceso a bases de datos

!!! warning "🚧 Contenido pendiente de desarrollo"
    Esta página todavía no tiene la teoría redactada. Usa el prompt de más abajo con `/improve-notes`, apoyándote en el proyecto **GameVault** adjunto, para generar el contenido definitivo.

---

## Prompt para `/improve-notes`

```text
Redacta el apartado de teoría "Conectores y protocolos de acceso a bases de datos" del
Tema 1 (RA2 - Manejo de conectores) del módulo Acceso a Datos (0486), semana real 3 del
calendario. Sigue las convenciones de estilo de docs/tema0/introduccion-docker.md y del
README.md del repo (tabs-colored, admonitions, densidad visual, pretérito perfecto
compuesto).

Criterios de evaluación de RA2 que cubre este apartado (curriculum.md):
- a) Ventajas e inconvenientes de utilizar conectores.
- b) Gestores de bases de datos embebidos e independientes.
- d) Establecimiento de la conexión.
- e) Definición de la estructura de la base de datos.

Contenidos de currículo a trabajar: el desfase objeto-relacional (impedance mismatch),
protocolos de acceso a bases de datos y conectores (JDBC como estándar en Java), pooling
de conexiones, y ejecución de sentencias de descripción de datos (DDL).

ESTRUCTURA OBLIGATORIA — teoría primero, proyecto después. El alumnado conoce SQL y
bases de datos relacionales de 1º de DAM, pero NADA de acceso a datos desde Java.

PARTE 1 — Teoría general, desde cero y sin mencionar GameVault todavía:
- El problema de partida: mi programa Java tiene objetos, mi base de datos tiene tablas;
  ¿cómo hablan entre sí? Presenta el desfase objeto-relacional con un ejemplo genérico
  mínimo (una clase Alumno con una lista de Asignatura frente a dos tablas con clave
  foránea): tipos que no coinciden, relaciones que son referencias en un lado y JOINs en
  el otro, identidad (== vs clave primaria).
- Qué es un protocolo de acceso a base de datos y qué es un conector/driver: el
  intérprete entre tu lenguaje y el gestor concreto; JDBC como la API estándar de Java
  (una interfaz común, un driver por gestor: PostgreSQL, MySQL...).
- Gestor embebido vs. independiente: definición de cada uno, con ejemplos (H2/SQLite vs.
  PostgreSQL/MySQL) y cuándo conviene cada cual.
- Qué es el pooling de conexiones y qué problema resuelve: abrir una conexión es caro;
  el pool las reutiliza — con una analogía sencilla (taquillas ya abiertas frente a
  forzar la cerradura cada vez).

PARTE 2 — Aterrizaje en GameVault (com.aleroig.gamevault):
- El `docker-compose.yaml` del proyecto levanta un PostgreSQL independiente (imagen
  `postgres:18-alpine`, puerto publicado en el host) — úsalo para explicar qué es un
  gestor de BD independiente frente a uno embebido (menciona H2 como contraste: un
  gestor embebido que podría añadirse en un perfil `test` para no depender de Docker en
  los tests unitarios — esto es una MEJORA sobre el estado actual del proyecto, que solo
  usa PostgreSQL, así que explícalo como algo que el alumnado podría incorporar).
- Nota breve a incluir aquí (para evitar confusión más adelante, en la Actividad 3.3):
  el propio proyecto usa DOS versiones distintas de la imagen de Postgres — `18-alpine`
  en `docker-compose.yaml` (el gestor de desarrollo, el que se levanta en esta semana) y
  `16-alpine` en el contenedor de Testcontainers de los tests de integración (Tema 3).
  Una frase basta: la versión del gestor de desarrollo y la versión usada en tests no
  tienen por qué coincidir exactamente, siempre que ambas sean compatibles con el SQL y
  las características (como JSONB) que usa el proyecto.
- La configuración de conexión vive en `application-dev.yaml` / `application.yaml`
  (datasource url, usuario, contraseña) — explica ahí el pooling (HikariCP, el pool por
  defecto de Spring Boot) sin que el alumnado tenga que configurarlo a mano.
- Las entidades `Videojuego` y `Estudio` (paquete `catalogo`) son el ejemplo de
  "definición de la estructura de la base de datos": explica `@Entity`, `@Table`,
  `@Id`/`@GeneratedValue`, la relación `@ManyToOne`/`@OneToMany` entre ambas, y la
  propiedad `spring.jpa.hibernate.ddl-auto` (qué hace `update` frente a `validate` o
  `none`, y por qué en un proyecto real no se usa `create-drop`).
- Explica el desfase objeto-relacional con el propio ejemplo Videojuego/Estudio: una
  clase Java con relaciones y colecciones frente a tablas con claves foráneas — por qué
  hace falta algo (un conector, luego un ORM) para tender ese puente.

No incluyas todavía el JDBC puro (Connection/Statement/ResultSet manual) ni los
procedimientos almacenados: esos apartados van en jdbc-puro.md y
procedimientos-almacenados.md, más adelante en este mismo tema. Este apartado se queda en
el nivel de "por qué existen los conectores y cómo se define la estructura de la BD",
justo antes de que en la Actividad 1.1 el alumnado levante su propio PostgreSQL con
Docker Compose y replique las entidades Videojuego/Estudio.
```
