<a id="consultas-jpql-transacciones"></a>

# 🧩 3. Consultas JPQL y transacciones

!!! warning "🚧 Contenido pendiente de desarrollo"
    Esta página todavía no tiene la teoría redactada. Usa el prompt de más abajo con
    `/improve-notes`, apoyándote en el proyecto **GameVault** adjunto, para generar el
    contenido definitivo. Este apartado cierra el RA3.

---

## Prompt para `/improve-notes`

```text
Redacta el apartado de teoría "Consultas JPQL y transacciones" del Tema 2 (RA3 - ORM) del
módulo Acceso a Datos (0486), semana real 9 del calendario — apartado que CIERRA el RA3.
Sigue las convenciones de estilo del README.md del repo.

Criterios de evaluación de RA3 que cubre este apartado (curriculum.md):
- f) Se han desarrollado aplicaciones que realizan consultas usando el lenguaje SQL
  (aquí, vía JPQL, el "SQL orientado a objetos" de JPA).
- g) Se han gestionado las transacciones.

ESTRUCTURA — teoría primero: antes del caso del proyecto, explica desde cero qué es
JPQL: un lenguaje de consultas de la especificación JPA que se parece a SQL pero opera
sobre ENTIDADES y sus PROPIEDADES, no sobre tablas y columnas. Muestra la misma consulta
genérica en SQL y en JPQL lado a lado para que la diferencia se vea (nombres de clase
con mayúscula, propiedades en vez de columnas, navegación por relaciones con el punto en
vez de JOIN explícito sobre claves). Explica también las tres vías de consulta de Spring
Data y cuándo usar cada una (naming de método para lo simple, Specifications para
filtros dinámicos — Tema 2, apartado 2 —, @Query/JPQL para consultas complejas fijas
como agregaciones), y qué es una agregación (GROUP BY/COUNT) por si el repaso de SQL lo
necesita.

Contenido central: `@Query` con JPQL en un método de repositorio, para consultas que no
se pueden expresar con Specifications ni con el naming de métodos de Spring Data (por
ejemplo, un ranking/agregación). Nota: en el estado actual de GameVault no existe todavía
un ranking de estudios implementado con `@Query` JPQL sobre EstudioRepository.java (que
hoy es un JpaRepository plano, sin métodos propios) — esto es una MEJORA que el
alumnado va a añadir, siguiendo el estilo del proyecto (mismo paquete `catalogo`, mismas
convenciones Lombok/@RequiredArgsConstructor que VideojuegoService.java).

Explica la sintaxis JPQL básica con un ejemplo guiado de "ranking de estudios por número
de videojuegos publicados" (algo como `SELECT e FROM Estudio e JOIN e.videojuegos v
GROUP BY e ORDER BY COUNT(v) DESC`, adaptado a que se pueda anotar con `@Query` en
EstudioRepository), contrastando JPQL con el SQL nativo (JPQL opera sobre entidades y
sus propiedades, no sobre nombres de tabla/columna).

Retoma la gestión de transacciones ya vista en el Tema 1 (`@Transactional` en
VideojuegoService) pero ahora desde el ángulo de las consultas: explica
`@Transactional(readOnly = true)` en operaciones de solo lectura (optimización, evita
locks innecesarios) frente a `@Transactional` a secas en operaciones de escritura, usando
VideojuegoService.java como referencia constante en todo el tema.

Cierra el apartado recapitulando todo RA3 en 3-4 frases (instalación/configuración del
ORM → mapeo con anotaciones → persistencia con Specifications → consultas JPQL) antes de
pasar al Tema 3 (RA4, bases de datos objeto-relacionales con JSONB), que retoma
`VideojuegoSpecifications.disponibleEnPlataforma` y la columna `detallesPlataforma` ya
mencionadas de pasada en persistencia-objetos.md.
```
