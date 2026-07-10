# 🧪 Actividad 5.3: Integración final del proyecto — cierre de RA6

!!! warning "🚧 Contenido pendiente de desarrollo"
    Esta actividad todavía no está redactada. Usa el prompt de más abajo con
    `/improve-notes`, apoyándote en el proyecto **GameVault** adjunto, para generar el
    enunciado definitivo. Es la actividad final del módulo.

---

## Prompt para `/improve-notes`

```text
Redacta la Actividad 5.3 del Tema 5 (RA6) del módulo Acceso a Datos (0486), semana real
19 del calendario — actividad que CIERRA el RA6 y es la ENTREGA FINAL de todo el módulo
0486. Sigue el patrón de estructura de docs/tema0/actividad_0_6.md y usa la skill
/actividad-plantilla-acceso-a-datos si necesita plantilla/solución en .docx.

IMPORTANTE — enfoque: es una PRÁCTICA GUIADA, no un reto. El alumnado trabaja sobre su
propia copia de GameVault (el mismo proyecto adjunto, construido individualmente durante
el curso). El enunciado debe guiar paso a paso, mostrando el código y explicando cada
decisión; solo se deja sin guiar, como mini-reto, lo que repita un patrón idéntico ya
mostrado en la misma actividad o en actividades anteriores.

Objetivo (RA6, criterios h, i — cierre del RA y del módulo): que el alumnado entregue su
GameVault completo (construido desde el Tema 1: catálogo en PostgreSQL con
JDBC/JPA/Specifications/JSONB, reviews en MongoDB, componentes desacoplados con
interfaces propias, más las MEJORAS añadidas: procedimiento almacenado, ranking JPQL,
borrado en cascada Postgres→Mongo, PUT de reseñas con autoría, ReviewsConsultaService)
con un test de integración final al estilo de
src/test/java/com/aleroig/gamevault/integration/GamevaultApiTest.java, que levante con
Testcontainers TODAS las bases de datos usadas (como mínimo PostgreSQL y MongoDB) y
verifique un flujo real de extremo a extremo.

Estructura sugerida de pasos guiados:
1. El test de integración de flujo completo, guiado por tramos: el enunciado da el
   esqueleto del test y guía el primer tramo (crear un estudio y un videojuego con
   detallesPlataforma vía la API, con el código mostrado); los tramos siguientes (crear
   una reseña en MongoDB para ese videojuego, consultar el resumen, y verificar el
   borrado en cascada de la Actividad 4.2 al eliminar el videojuego) son mini-retos que
   repiten patrones de test ya escritos en las actividades 3.3 y anteriores — se indica
   qué verificar en cada tramo, no el código.
2. Un documento breve de cierre guiado por un guion dado (README del GameVault propio, o
   sección de memoria si el centro lo pide así): listar los componentes desarrollados y
   su responsabilidad, 1-2 frases por componente, siguiendo el espíritu de
   docs/04-decisiones-arquitectura.md de la referencia.
3. Una autoevaluación razonada del propio alumnado: de los RA trabajados en el módulo
   (RA2 a RA6; RA1 se ha trabajado en la FCT), ¿en cuál se sienten más flojos y por qué?
   Respuesta concreta y personal, no genérica, como parte de la entrega.
4. Verificación guiada de que el CI de su GameVault (configurado desde el Tema 0)
   ejecuta correctamente estos tests de integración en cada push (qué mirar en la
   pestaña Actions, indicado en el enunciado), cerrando el círculo con lo aprendido en
   Git/GitHub Actions al principio del curso.

Esta es la última actividad evaluable del módulo: el enunciado debe transmitir que es un
cierre, no una tarea más, y debe permitir a cada alumno mostrar cómo ha evolucionado su
GameVault durante el curso.
```
