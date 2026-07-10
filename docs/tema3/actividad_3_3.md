# 🧪 Actividad 3.3: Pruebas de integración sobre JSONB — cierre de RA4

!!! warning "🚧 Contenido pendiente de desarrollo"
    Esta actividad todavía no está redactada. Usa el prompt de más abajo con
    `/improve-notes`, apoyándote en el proyecto **GameVault** adjunto, para generar el
    enunciado definitivo.

---

## Prompt para `/improve-notes`

```text
Redacta la Actividad 3.3 del Tema 3 (RA4) del módulo Acceso a Datos (0486), semana real
12 del calendario — actividad que CIERRA el RA4. Sigue el patrón de estructura de
docs/tema0/actividad_0_6.md y usa la skill /actividad-plantilla-acceso-a-datos si
necesita plantilla/solución en .docx.

IMPORTANTE — enfoque: es una PRÁCTICA GUIADA, no un reto. El alumnado trabaja sobre su
propia copia de GameVault (el mismo proyecto adjunto, construido individualmente durante
el curso). El enunciado debe guiar paso a paso, mostrando el código y explicando cada
decisión; solo se deja sin guiar, como mini-reto, lo que repita un patrón idéntico ya
mostrado en la misma actividad.

Objetivo (RA4, criterio h — cierre del RA): que el alumnado escriba en su GameVault,
guiado, un test de integración con Testcontainers que verifique el comportamiento real
de la columna JSONB (Actividad 3.1) y de la Specification `jsonb_exists` (Actividad
3.2), siguiendo el patrón de
src/test/java/com/aleroig/gamevault/integration/GamevaultApiTest.java:
`@Testcontainers`, `@Container` + `@ServiceConnection` sobre un `PostgreSQLContainer`,
`@AutoConfigureMockMvc`, `@ActiveProfiles("test")`.

Estructura sugerida de pasos guiados:
1. Configuración guiada del test: dependencias de Testcontainers en el pom, la clase de
   test con sus anotaciones y el contenedor de PostgreSQL (imagen como en la referencia,
   `postgres:16-alpine`), todo con el código mostrado y explicado — incluida la
   explicación de qué hace `@ServiceConnection` y por qué el test no depende del Docker
   Compose de desarrollo.
2. Primer test guiado al completo: crear un videojuego con `detallesPlataforma` vía la
   API (MockMvc, `POST`) y comprobar con un `GET` filtrado por plataforma que aparece —
   código completo mostrado y explicado línea a línea (jsonPath, andExpect...).
3. Mini-reto (repite el patrón del paso 2): el test negativo — filtrar por una
   plataforma que el videojuego NO tiene y comprobar que no aparece. Solo se indica el
   objetivo; la estructura es idéntica.
4. Pregunta de comprensión: ¿qué detectaría este test con Testcontainers que NO
   detectaría un test con mocks del repositorio? Orientar hacia un error concreto de
   mapeo JSONB que solo se vería contra un PostgreSQL real.
5. Cierre del RA4: un resumen propio del alumnado (5-6 líneas) de cómo su GameVault ha
   evolucionado a lo largo de las tres actividades del tema, desde "columna JSONB
   básica" hasta "consulta filtrada y probada con Testcontainers". Incluye aquí,
   explícitamente como parte del entregable (evidencia del criterio b, "no aplica" —
   ver nota de profesorado en tema3/index.md), 2-3 líneas explicando por qué JSONB no
   abre ni cierra una conexión propia: reutiliza la misma conexión JDBC/JPA ya
   gestionada por el framework desde el Tema 1.

Esta actividad cierra RA4 y da paso al Tema 4 (RA5, MongoDB), donde se introduce por
primera vez una base de datos NoSQL distinta de PostgreSQL.
```
