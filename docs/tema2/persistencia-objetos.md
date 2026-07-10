<a id="persistencia-objetos"></a>

# 🧩 2. Persistencia y recuperación de objetos

!!! warning "🚧 Contenido pendiente de desarrollo"
    Esta página todavía no tiene la teoría redactada. Usa el prompt de más abajo con
    `/improve-notes`, apoyándote en el proyecto **GameVault** adjunto, para generar el
    contenido definitivo.

---

## Prompt para `/improve-notes`

```text
Redacta el apartado de teoría "Persistencia y recuperación de objetos" del Tema 2 (RA3 -
ORM) del módulo Acceso a Datos (0486), semana real 8 del calendario. Sigue las
convenciones de estilo del README.md del repo.

Criterios de evaluación de RA3 que cubre este apartado (curriculum.md):
- d) Se han aplicado mecanismos de persistencia a los objetos.
- e) Se han desarrollado aplicaciones que modifican y recuperan objetos persistentes.

ESTRUCTURA OBLIGATORIA — teoría primero, proyecto después.

PARTE 1 — Teoría general, desde cero:
- El patrón repositorio como concepto: una interfaz que representa "el almacén de
  objetos de un tipo" (guardar, buscar por id, listar, borrar), independiente de cómo se
  implemente por debajo — y qué aporta Spring Data JPA: tú declaras la interfaz, él
  genera la implementación.
- Las operaciones básicas de persistencia sobre un ejemplo genérico: save() (¿inserta o
  actualiza? — según si el objeto tiene id), findById() y el problema del "puede no
  existir" (qué es Optional y cómo se usa orElseThrow — el alumnado no lo conoce),
  findAll(), delete().
- Qué es una consulta dinámica y por qué las consultas fijas se quedan cortas: el caso
  universal del buscador con N filtros opcionales (cada usuario rellena unos campos u
  otros) y la explosión de combinaciones si se intenta resolver con métodos fijos.
  Presenta la idea de Specification/Criteria: construir la consulta como piezas
  combinables en tiempo de ejecución.
- Qué es la paginación y por qué ninguna API seria devuelve "todo": página, tamaño,
  orden — el concepto, antes de la clase Pageable concreta.

PARTE 2 — Aterrizaje en GameVault (com.aleroig.gamevault):
- com/aleroig/gamevault/catalogo/VideojuegoRepository.java: interfaz que extiende
  `JpaRepository<Videojuego, Long>` y además `JpaSpecificationExecutor<Videojuego>` —
  explica que con JpaRepository el alumnado ya tiene save/findById/findAll "gratis", sin
  escribir una sola línea de SQL ni de JDBC (contraste directo con el Tema 1).
- com/aleroig/gamevault/catalogo/VideojuegoSpecifications.java: repásalo método a
  método (`tituloContiene`, `precioMayorOIgualA`, `precioMenorOIgualA`,
  `perteneceAlEstudio`, `disponibleEnPlataforma`) como ejemplo de "consulta que se
  construye dinámicamente combinando condiciones opcionales" — explica el patrón
  `Specification.unrestricted()` cuando el filtro es null (no añade condición) y cómo se
  combinan con `.and(...)` en VideojuegoService.findAllPaginated(). Puedes mencionar de
  pasada `disponibleEnPlataforma` (que usa la función `jsonb_exists` sobre la columna
  JSONB) sin profundizar en JSONB todavía — eso es el Tema 3 (RA4) completo.
- com/aleroig/gamevault/catalogo/VideojuegoService.java (método findAllPaginated):
  muestra cómo se combinan varias Specifications con `Specification.where(...).and(...)`
  y se pasan a `videojuegoRepository.findAll(spec, pageable)` — un ejemplo real de
  recuperación de objetos persistentes con filtros dinámicos y paginación.

Explica también `update()` en VideojuegoService (localizar con findById, modificar los
campos del objeto managed, volver a hacer save) como ejemplo de "modificar y recuperar
objetos persistentes", y contrapón brevemente con el `INSERT`/`UPDATE` manual que el
alumnado ya escribió a mano en el Tema 1. No entres en `@Query` JPQL todavía: eso es el
siguiente apartado, consultas-jpql-transacciones.md.
```
