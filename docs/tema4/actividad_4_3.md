# 🧪 Actividad 4.3: PUT de reseñas con control de autoría — cierre de RA5

!!! warning "🚧 Contenido pendiente de desarrollo"
    Esta actividad todavía no está redactada. Usa el prompt de más abajo con
    `/improve-notes`, apoyándote en el proyecto **GameVault** adjunto, para generar el
    enunciado definitivo.

---

## Prompt para `/improve-notes`

```text
Redacta la Actividad 4.3 del Tema 4 (RA5 - BD documentales) del módulo Acceso a Datos
(0486), semana real 16 del calendario (18-24 enero) — actividad que CIERRA el RA5. Sigue
el patrón de estructura de docs/tema0/actividad_0_6.md y usa la skill
/actividad-plantilla-acceso-a-datos si necesita plantilla/solución en .docx.

IMPORTANTE — enfoque: es una PRÁCTICA GUIADA, no un reto. El alumnado trabaja sobre su
propia copia de GameVault (el mismo proyecto adjunto, construido individualmente durante
el curso). El enunciado debe guiar paso a paso, mostrando el código y explicando cada
decisión; solo se deja sin guiar, como mini-reto, lo que repita un patrón idéntico ya
mostrado en la misma actividad.

Objetivo (RA5, criterio e — cierre del RA): que el alumnado añada, guiado, un `PUT
/api/v1/videojuegos/{videojuegoId}/reviews/{reviewId}` a su controlador de reseñas
(VideojuegoReviewController, que en la referencia adjunta solo tiene GET, POST y
GET .../resumen — este PUT con control de autoría es una MEJORA sobre el proyecto
adjunto): solo el autor original de la reseña puede modificarla.

Estructura sugerida de pasos guiados:
1. El endpoint PUT guiado al completo, código mostrado y explicado: recibe el
   `Principal` igual que ya hace el POST existente (`principal.getName()`), carga el
   documento por id desde MongoDB, compara el autor guardado con el usuario autenticado,
   y devuelve 403 si no coincide o 404 si la reseña no existe.
2. La actualización real (puntuación y/o comentario) usando `save()` sobre el documento
   ya cargado y modificado, explicando en el enunciado que `save()` sirve tanto para
   crear como para actualizar en Spring Data MongoDB (mismo concepto ya visto con JPA).
3. Prueba guiada con dos usuarios distintos (peticiones de ejemplo dadas): uno modifica
   su propia reseña (debe funcionar) y otro intenta modificar la reseña ajena (debe
   recibir 403) — documentar el resultado observado de cada caso.
4. Nota para quien redacte: en el calendario unificado, PSP ya ha cubierto su RA5 de
   seguridad (JWT, roles — semanas reales 7-11) antes de esta semana 16, así que el
   GameVault del alumnado ya tiene autenticación real con Principal funcionando; da por
   hecho ese estado. Solo si algún alumno va retrasado con PSP, indica como plan B
   temporal recibir el "autor actual" en el DTO, marcándolo explícitamente como
   simplificación provisional.
5. Un cierre de RA5 (5-6 líneas propias del alumnado) que repase las tres actividades del
   tema: de "conectar y consultar" a "modificar con control de acceso".

Esta actividad cierra el Tema 4 y da paso al Tema 5 (RA6, componentes de acceso a datos),
que retomará CatalogoConsultaService como ejemplo central de componente reutilizable.
```
