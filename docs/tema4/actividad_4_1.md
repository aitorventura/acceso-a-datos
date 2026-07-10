# 🧪 Actividad 4.1: Reseñas de videojuegos en MongoDB

!!! warning "🚧 Contenido pendiente de desarrollo"
    Esta actividad todavía no está redactada. Usa el prompt de más abajo con
    `/improve-notes`, apoyándote en el proyecto **GameVault** adjunto, para generar el
    enunciado definitivo.

---

## Prompt para `/improve-notes`

```text
Redacta la Actividad 4.1 del Tema 4 (RA5 - BD documentales) del módulo Acceso a Datos
(0486), semana real 14 del calendario (14-20 diciembre). Sigue el patrón de estructura de
docs/tema0/actividad_0_6.md y usa la skill /actividad-plantilla-acceso-a-datos si
necesita plantilla/solución en .docx.

IMPORTANTE — enfoque: es una PRÁCTICA GUIADA, no un reto. El alumnado trabaja sobre su
propia copia de GameVault (el mismo proyecto adjunto, construido individualmente durante
el curso). El enunciado debe guiar paso a paso, mostrando el código y explicando cada
decisión; solo se deja sin guiar, como mini-reto, lo que repita un patrón idéntico ya
mostrado en la misma actividad.

Objetivo (RA5, criterios b, c): que el alumnado construya en su GameVault, guiado, el
módulo `reviews` tal y como existe en com/aleroig/gamevault/reviews/: levantar MongoDB
con Docker Compose (nuevo servicio junto al PostgreSQL ya existente), crear la entidad
`Review` (`@Document`, id String, campo de relación lógica videojuegoId) y el
repositorio `ReviewRepository extends MongoRepository<Review, String>` con
`findByVideojuegoId` por naming de método, sin escribir consultas explícitas.

Estructura sugerida de pasos guiados:
1. Añadir el servicio de MongoDB al docker-compose.yaml (fragmento dado y comentado,
   imagen oficial `mongo`) y la configuración de conexión en application-dev.yaml, más
   la dependencia spring-boot-starter-data-mongodb en el pom — todo guiado.
2. La entidad Review y el ReviewRepository, guiados con el código de la referencia
   mostrado y explicado (`@Document`, por qué el id es String, qué genera Spring Data a
   partir del nombre `findByVideojuegoId`).
3. El patrón de "integridad referencial manual" visto en la teoría, guiado: el
   ReviewService comprueba vía CatalogoConsultaService que el videojuego existe en
   PostgreSQL antes de guardar o consultar en Mongo, devolviendo 404 si no — código de
   la referencia (ReviewService.java) mostrado y explicado. Pregunta de comprensión:
   ¿por qué esta comprobación no la puede hacer MongoDB por sí solo?
4. El GET de reseñas por videojuego y el POST, guiados (VideojuegoReviewController de la
   referencia; si la autenticación con Principal aún no está en su GameVault en este
   punto, indicar la simplificación temporal de recibir el autor en el DTO).
5. Mini-reto (mismo patrón que el paso 4): el endpoint de resumen
   `GET /api/v1/videojuegos/{id}/reviews/resumen` — dar el DTO (ReviewResumenDTO: total
   y puntuación media, calculados en memoria con streams como en la referencia) y dejar
   que completen el service y el endpoint solos.
6. Experimento guiado de cierre: borrar un videojuego de PostgreSQL que tenga reseñas y
   observar que estas siguen en MongoDB ("reseñas huérfanas") — pasos dados; describir
   el problema observado como preparación de la Actividad 4.2.

Esta actividad no debe entrar todavía en creación/borrado de colecciones completas ni en
el PUT de reseñas con control de autoría: eso son las actividades 4.2 y 4.3.
```
