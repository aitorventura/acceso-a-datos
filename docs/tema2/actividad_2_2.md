# 🧪 Actividad 2.2: Consultas dinámicas con Specifications

!!! warning "🚧 Contenido pendiente de desarrollo"
    Esta actividad todavía no está redactada. Usa el prompt de más abajo con
    `/improve-notes`, apoyándote en el proyecto **GameVault** adjunto, para generar el
    enunciado definitivo.

---

## Prompt para `/improve-notes`

```text
Redacta la Actividad 2.2 del Tema 2 (RA3 - ORM) del módulo Acceso a Datos (0486), semana
real 8 del calendario. Sigue el patrón de estructura de docs/tema0/actividad_0_6.md y
usa la skill /actividad-plantilla-acceso-a-datos si necesita plantilla/solución en .docx.

IMPORTANTE — enfoque: es una PRÁCTICA GUIADA, no un reto. El alumnado trabaja sobre su
propia copia de GameVault (el mismo proyecto adjunto, construido individualmente durante
el curso). El enunciado debe guiar paso a paso, mostrando el código y explicando cada
decisión; solo se deja sin guiar, como mini-reto, lo que repita un patrón idéntico ya
mostrado en la misma actividad.

Objetivo (RA3, criterios d, e): que el alumnado construya en su GameVault, guiado, el
sistema de filtros dinámicos del listado de videojuegos, tal y como existe en la
referencia: com/aleroig/gamevault/catalogo/VideojuegoSpecifications.java +
VideojuegoRepository.java (extendiendo JpaSpecificationExecutor) +
VideojuegoService.findAllPaginated() + VideojuegoFiltroDTO.

Estructura sugerida de pasos guiados:
1. Preparación guiada: hacer que VideojuegoRepository extienda JpaSpecificationExecutor
   y crear la clase VideojuegoSpecifications (explicando qué es una Specification y el
   patrón `Specification.unrestricted()` cuando el filtro es null).
2. Primera Specification guiada al completo, código mostrado y explicado:
   `tituloContiene` (con el `criteriaBuilder.like` + `lower`).
3. Segunda Specification guiada: `precioMayorOIgualA` (nuevo matiz:
   `greaterThanOrEqualTo` sobre BigDecimal).
4. Mini-retos (repiten el patrón ya visto dos veces): `precioMenorOIgualA` y
   `perteneceAlEstudio` — solo se indica qué debe filtrar cada una; la estructura es
   idéntica a las anteriores. (NO incluyas aquí `disponibleEnPlataforma`: usa jsonb y
   llega en el Tema 3.)
5. Combinación guiada en el service con `Specification.where(...).and(...)` +
   paginación con `Pageable`, y el DTO de filtro VideojuegoFiltroDTO, replicando
   findAllPaginated() de la referencia con el código mostrado.
6. Verificación guiada con peticiones reales dadas en el enunciado: con filtros, sin
   filtros (todos null deben devolver todo), y combinando varios.
7. Pregunta de comprensión: ¿por qué conviene devolver `Specification.unrestricted()`
   cuando un filtro es null, en vez de no añadir el `.and(...)` con un if fuera del
   método?

Esta actividad no debe usar `@Query` JPQL (eso es la Actividad 2.3, que cierra el RA3);
el foco es exclusivamente Specifications dinámicas y paginación con `Pageable`.
```
