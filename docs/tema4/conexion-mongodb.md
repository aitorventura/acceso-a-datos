<a id="conexion-mongodb"></a>

# 🧩 1. Conexión a MongoDB y consultas sobre reseñas

!!! warning "🚧 Contenido pendiente de desarrollo"
    Esta página todavía no tiene la teoría redactada. Usa el prompt de más abajo con
    `/improve-notes`, apoyándote en el proyecto **GameVault** adjunto, para generar el
    contenido definitivo.

---

## Prompt para `/improve-notes`

```text
Redacta el apartado de teoría "Conexión a MongoDB y consultas sobre reseñas" del Tema 4
(RA5 - BD documentales) del módulo Acceso a Datos (0486), semana real 14 del calendario
(14-20 diciembre). Sigue las convenciones de estilo del README.md del repo.

Criterios de evaluación de RA5 que cubre este apartado (curriculum.md):
- b) Se ha establecido la conexión con la base de datos.
- c) Se han desarrollado aplicaciones que efectúan consultas sobre el contenido de la
  base de datos.

ESTRUCTURA OBLIGATORIA — teoría primero, proyecto después.

PARTE 1 — Teoría general, desde cero:
- Qué es NoSQL como familia: bases de datos que abandonan el modelo tabla/fila/SQL en
  favor de otros modelos de datos; menciona en una frase cada tipo principal
  (documentales, clave-valor, columnares, de grafos) y sitúa este tema en las
  documentales.
- Qué es una base de datos documental nativa: sus dos piezas — el DOCUMENTO (un objeto
  completo tipo JSON, autocontenido, con estructura propia que puede variar de un
  documento a otro) y la COLECCIÓN (un grupo de documentos, sin esquema fijo impuesto).
  Contrasta explícitamente con lo relacional: no hay tablas, ni columnas, ni claves
  foráneas, ni JOINs. Y con el JSONB del Tema 3: allí el JSON era UNA columna dentro de
  una tabla relacional; aquí TODO es documento.
- Qué es MongoDB: el gestor documental más usado; documentos BSON (JSON binario), el id
  como ObjectId alfanumérico, y cómo se organiza (servidor → bases de datos →
  colecciones → documentos). Muestra un documento de ejemplo genérico y la operación de
  consulta equivalente a un SELECT sencillo en la sintaxis de mongosh, para que se vea
  la forma de trabajar antes de que Spring la esconda.
- Cuándo encaja lo documental y cuándo no (el criterio a se profundiza en el apartado
  2, pero deja aquí la intuición): datos autocontenidos y de estructura variable sí;
  relaciones fuertes y transacciones complejas entre entidades, mejor relacional.

PARTE 2 — Aterrizaje en GameVault (com.aleroig.gamevault, paquete `reviews`), con
Spring Data MongoDB como la capa que conecta MongoDB con Spring Boot de forma análoga a
Spring Data JPA con PostgreSQL. Arranca explicando por qué GameVault usa DOS bases de
datos distintas para dos partes del mismo dominio:
- com/aleroig/gamevault/reviews/Review.java: `@Document(collection = "review")`, `@Id`
  de tipo String (a diferencia del Long autogenerado de las entidades JPA — explica por
  qué en Mongo los IDs son alfanuméricos, tipo ObjectId), y los campos `videojuegoId`,
  `autor`, `puntuacion`, `comentario` — remarca que `videojuegoId` es una "relación
  lógica" con el catálogo en PostgreSQL, NO una clave foránea real (no hay integridad
  referencial automática entre dos motores de base de datos distintos).
- com/aleroig/gamevault/reviews/ReviewRepository.java: `MongoRepository<Review, String>`
  con `findByVideojuegoId(Long videojuegoId)` — muestra que Spring Data también genera
  consultas automáticas por naming de método en MongoDB, igual que hacía con
  JpaRepository en el Tema 2, sin que el alumnado escriba una query explícita.
- com/aleroig/gamevault/reviews/ReviewService.java (método findByVideojuegoId): fíjate en
  el comentario "1. Verificamos que el juego exista en PostgreSQL antes de buscar sus
  reseñas" — usa `CatalogoConsultaService.existeVideojuego(...)` (del paquete
  `catalogo.api`) para comprobar en PostgreSQL antes de consultar Mongo. Explica esto como
  el patrón central de "integridad referencial manual" cuando dos módulos usan motores de
  BD distintos: ya que Mongo no puede validar una clave foránea hacia una tabla de
  Postgres, el propio código de la aplicación tiene que hacerlo.
- com/aleroig/gamevault/reviews/dto/ReviewResumenDTO.java y el método
  getResumenByVideojuegoId de ReviewService.java: ejemplo de consulta que agrega datos en
  memoria (total de reseñas, puntuación media) tras traer los documentos de Mongo — no
  usa el framework de agregación de Mongo (`$group`, `$avg`); coméntalo como una
  simplificación deliberada y como posible mejora futura del propio alumnado si quiere ir
  más allá.

Sobre "establecer la conexión" (criterio b): menciona brevemente
`spring-boot-starter-data-mongodb` en el pom.xml y la propiedad de conexión REAL del
proyecto — `spring.mongodb.uri` en application-dev.yaml (OJO: en este proyecto
`mongodb` cuelga directamente de `spring`, NO de `spring.data`; comprueba el YAML real
antes de redactar para no enseñar una ruta de propiedad que no existe en el fichero que
el alumnado tiene delante), en paralelo al datasource de PostgreSQL ya visto en el Tema 1.

No entres todavía en creación/borrado de colecciones ni en el contraste
relacional/documental en profundidad: eso es el siguiente apartado,
colecciones-documentales.md.
```
