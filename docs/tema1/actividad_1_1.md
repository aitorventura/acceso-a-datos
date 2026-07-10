# 🧪 Actividad 1.1: Docker Compose + PostgreSQL — arranque de las entidades del catálogo

!!! warning "🚧 Contenido pendiente de desarrollo"
    Esta actividad todavía no está redactada. Usa el prompt de más abajo con
    `/improve-notes`, apoyándote en el proyecto **GameVault** adjunto y en el patrón ya
    usado en `docs/tema0/actividad_0_6.md` (plantilla/solución en `.docx`), para generar
    el enunciado definitivo.

---

## Prompt para `/improve-notes`

```text
Redacta la Actividad 1.1 del Tema 1 (RA2 - Manejo de conectores) del módulo Acceso a
Datos (0486), semana real 3 del calendario. Sigue el patrón de
docs/tema0/actividad_0_6.md (misma estructura de encabezado y estilo de pasos guiados)
y usa la skill /actividad-plantilla-acceso-a-datos para generar la plantilla en
blanco y la solución en .docx si la actividad lo requiere.

IMPORTANTE — enfoque: es una PRÁCTICA GUIADA, no un reto. El alumnado desarrolla su
propia copia de GameVault (el mismo proyecto adjunto: cada alumno construye el suyo
individualmente durante el curso, y el repositorio adjunto es la referencia final del
profesor). El enunciado debe guiar paso a paso, mostrando el código y explicando cada
decisión; solo se deja sin guiar, como mini-reto final, lo que repita un patrón idéntico
ya mostrado en la misma actividad o en la teoría.

Objetivo (RA2, criterios b, d, e): que el alumnado levante con Docker Compose el
PostgreSQL de su GameVault (gestor de BD independiente) y cree, guiado paso a paso, las
entidades `Videojuego` y `Estudio` tal y como existen en la referencia:

- com/aleroig/gamevault/catalogo/Videojuego.java (id, titulo, precio, fechaLanzamiento,
  relación @ManyToOne con Estudio, columna JSONB detallesPlataforma — para esta actividad
  el alumnado puede omitir el JSONB, que se trabaja más adelante en el Tema 3/RA4).
- com/aleroig/gamevault/catalogo/Estudio.java (id, nombre, pais, relación @OneToMany con
  Videojuego, cascade ALL + orphanRemoval).
- El docker-compose.yaml real del proyecto (servicio postgres, imagen postgres:18-alpine,
  variables de entorno POSTGRES_DB/USER/PASSWORD, puerto publicado) como referencia de
  cómo levantar el gestor.
- application-dev.yaml para la configuración del datasource y ddl-auto=update.

Estructura la práctica como pasos guiados:
0. PASO PREVIO obligatorio y explícito (no lo des por hecho): crear el proyecto Spring
   Boot vacío del alumnado desde cero, ANTES de tocar Docker Compose. Guía con
   Spring Initializr (start.spring.io, o el asistente equivalente del IDE): Maven, Java,
   la versión de Spring Boot del proyecto de referencia, y solo las dependencias mínimas
   para esta actividad (Spring Web — starter real `spring-boot-starter-webmvc`, Spring
   Data JPA, el driver de PostgreSQL, y Lombok). Explica qué genera el asistente (el
   `pom.xml` con esos starters, la clase `@SpringBootApplication`, la estructura de
   carpetas `src/main/java` / `src/main/resources`) y cómo se importa en el IDE. Aclara
   que el resto de dependencias del proyecto de referencia (seguridad, MongoDB, RabbitMQ,
   Redis, OpenAPI...) se irán añadiendo actividad a actividad, cuando toque cada una — no
   se añaden todas de golpe aquí. Este paso es la primera vez que el alumnado tiene un
   proyecto propio: sin él, no hay dónde levantar el Docker Compose ni crear las
   entidades de los pasos siguientes.
1. Levantar el PostgreSQL con Docker Compose (reutilizando lo aprendido en la Actividad
   0.6, pero ahora con una base de datos relacional, no una aplicación completa) — dar el
   fragmento de compose comentado línea a línea.
2. Crear la entidad Estudio guiada al completo (código mostrado y explicado), y después
   la entidad Videojuego con su relación @ManyToOne — de esta segunda puedes mostrar el
   esqueleto y dejar como mini-reto los campos que repiten el patrón ya visto (id
   autogenerado, columnas simples), guiando en cambio lo nuevo (la relación
   bidireccional).
3. Verificar con ddl-auto=update (o una herramienta como pgAdmin/DBeaver/psql) que
   Hibernate ha creado las tablas y la clave foránea esperadas — con los comandos/capturas
   esperados indicados en el enunciado.
4. Una pregunta de comprensión final (por ejemplo: ¿por qué cascade ALL + orphanRemoval
   en Estudio->Videojuego y no al revés?) para consolidar, no para bloquear el avance.

No incluyas CRUD completo, JDBC puro ni procedimientos almacenados: eso corresponde a las
actividades 1.2, 1.3 y 1.4. Esta actividad se limita a crear el proyecto, levantar el
gestor y definir la estructura de la base de datos (crit. b, d, e).
```
