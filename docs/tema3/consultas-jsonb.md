<a id="consultas-jsonb"></a>

# 🧩 2. Consultas sobre columnas JSONB

!!! warning "🚧 Contenido pendiente de desarrollo"
    Esta página todavía no tiene la teoría redactada. Usa el prompt de más abajo con
    `/improve-notes`, apoyándote en el proyecto **GameVault** adjunto, para generar el
    contenido definitivo.

---

## Prompt para `/improve-notes`

```text
Redacta el apartado de teoría "Consultas sobre columnas JSONB" del Tema 3 (RA4 - BD
objeto relacionales y orientadas a objetos) del módulo Acceso a Datos (0486), semana real
11 del calendario. Sigue las convenciones de estilo del README.md del repo.

Criterios de evaluación de RA4 que cubre este apartado (curriculum.md):
- e) Aplicaciones que realizan consultas.
- f) Modificación de los objetos almacenados.
- g) Gestión de las transacciones.

ESTRUCTURA — teoría primero: antes del código del proyecto, explica desde cero cómo se
consulta el contenido de un JSON almacenado en una base de datos: el problema (la
condición del WHERE ya no es sobre una columna, sino sobre una CLAVE dentro del objeto)
y las herramientas que ofrece PostgreSQL en SQL puro — muestra 2-3 ejemplos genéricos en
SQL directo con los operadores/funciones jsonb más comunes (jsonb_exists o el operador
`?` para "existe la clave", `->`/`->>` para extraer valores) sobre una tabla de ejemplo,
de modo que el alumnado entienda la consulta en SQL ANTES de verla envuelta en la
Criteria API de Java.

Después, aterriza en GameVault (com.aleroig.gamevault):
- com/aleroig/gamevault/catalogo/VideojuegoSpecifications.java, método
  `disponibleEnPlataforma(String plataforma)`: analízalo en detalle — usa
  `criteriaBuilder.function("jsonb_exists", Boolean.class,
  root.get("detallesPlataforma"), criteriaBuilder.literal(plataforma.toLowerCase()))`
  para invocar la función nativa `jsonb_exists` de PostgreSQL desde una Specification de
  Criteria API. Explica qué hace `jsonb_exists(columna, clave)` en PostgreSQL puro
  (comprobar si una clave de primer nivel existe en el JSON) antes de mostrar cómo se
  invoca desde Java.
- Contrasta esto con las Specifications "normales" ya vistas en el Tema 2
  (`tituloContiene`, `precioMayorOIgualA`) que operan sobre columnas relacionales
  corrientes con `criteriaBuilder.like`/`greaterThanOrEqualTo` — aquí, en cambio, hace
  falta una función específica del motor de base de datos porque el contenido vive dentro
  de un JSON, no en una columna con su propio tipo.
- com/aleroig/gamevault/catalogo/VideojuegoService.java (findAllPaginated): muestra cómo
  `disponibleEnPlataforma` se combina con el resto de Specifications igual que cualquier
  otra, con `.and(...)` — remarca que, desde el punto de vista de quien usa el
  repositorio, consultar JSONB o una columna normal es transparente gracias a
  Specification.

Sobre modificación de objetos JSONB (criterio f): retoma `update()` en
VideojuegoService.java y explica que actualizar `detallesPlataforma` reemplaza el Map
completo (ya visto en la Actividad 3.1) — aquí profundiza en por qué NO existe, en
Hibernate/JPA estándar, una forma directa de "hacer un merge parcial" del JSON sin traer
el objeto completo primero.

Sobre transacciones (criterio g): recuerda que `@Transactional` funciona exactamente
igual sobre entidades con columnas JSONB que sobre cualquier otra entidad — no hay nada
especial que gestionar aquí, y merece la pena decirlo explícitamente para no generar
falsas expectativas de complejidad añadida.

No entres en BD orientadas a objetos puras ni en el cierre de RA4: eso es el siguiente
apartado, bd-orientadas-a-objetos.md.
```
