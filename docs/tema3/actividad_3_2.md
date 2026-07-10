# 🧪 Actividad 3.2: Filtrar por plataforma con `jsonb_exists`

!!! warning "🚧 Contenido pendiente de desarrollo"
    Esta actividad todavía no está redactada. Usa el prompt de más abajo con
    `/improve-notes`, apoyándote en el proyecto **GameVault** adjunto, para generar el
    enunciado definitivo.

---

## Prompt para `/improve-notes`

```text
Redacta la Actividad 3.2 del Tema 3 (RA4 - BD objeto relacionales y orientadas a objetos)
del módulo Acceso a Datos (0486), semana real 11 del calendario. Sigue el patrón de
estructura de docs/tema0/actividad_0_6.md y usa la skill
/actividad-plantilla-acceso-a-datos si necesita plantilla/solución en .docx.

IMPORTANTE — enfoque: es una PRÁCTICA GUIADA, no un reto. El alumnado trabaja sobre su
propia copia de GameVault (el mismo proyecto adjunto, construido individualmente durante
el curso). El enunciado debe guiar paso a paso, mostrando el código y explicando cada
decisión; solo se deja sin guiar, como mini-reto, lo que repita un patrón idéntico ya
mostrado en la misma actividad.

Objetivo (RA4, criterios e, f, g): que el alumnado añada a su GameVault, guiado, la
Specification `disponibleEnPlataforma` tal y como existe en
com/aleroig/gamevault/catalogo/VideojuegoSpecifications.java, que usa `jsonb_exists`
sobre la columna JSONB creada en la Actividad 3.1, y la combine con el resto de filtros
que ya tiene en su servicio desde la Actividad 2.2.

Estructura sugerida de pasos guiados:
1. La Specification con `criteriaBuilder.function("jsonb_exists", ...)`, con el código de
   la referencia mostrado y explicado parte a parte (por qué hace falta `function` en vez
   de los métodos normales de criteriaBuilder, qué hace el `literal`, por qué el
   toLowerCase).
2. Añadir el campo `plataforma` al DTO de filtro y encadenar la nueva Specification con
   `.and(...)` en el service — mini-reto: es exactamente el mismo patrón que ya
   repitieron en la Actividad 2.2, así que basta con indicarlo sin dar el código.
3. Prueba guiada con peticiones dadas: filtrar por una plataforma que existe en los datos
   de la Actividad 3.1 y por una que no, comprobando ambos resultados.
4. Experimento guiado (criterio f, modificación de objetos): actualizar un videojuego
   quitando una plataforma de su Map (petición PUT de ejemplo dada) y comprobar que el
   filtro deja de devolverlo.
5. Pregunta de comprensión: ¿qué haría falta para filtrar por "disponible en Steam Y en
   Switch a la vez"? Orientar hacia `jsonb_exists_all`/`jsonb_exists_any` o dos
   `jsonb_exists` encadenados con `.and(...)`, pidiendo que expliquen la diferencia
   entre ambos enfoques (comprensión, no implementación obligatoria).

Esta actividad da paso a la Actividad 3.3, que cierra RA4 con pruebas de integración
sobre todo lo visto en el tema (persistencia + consultas JSONB).
```
