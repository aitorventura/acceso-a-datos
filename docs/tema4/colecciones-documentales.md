<a id="colecciones-documentales"></a>

# 🧩 2. Colecciones documentales: relacional vs. documental

!!! warning "🚧 Contenido pendiente de desarrollo"
    Esta página todavía no tiene la teoría redactada. Usa el prompt de más abajo con
    `/improve-notes`, apoyándote en el proyecto **GameVault** adjunto, para generar el
    contenido definitivo.

---

## Prompt para `/improve-notes`

```text
Redacta el apartado de teoría "Colecciones documentales: relacional vs. documental" del
Tema 4 (RA5 - BD documentales) del módulo Acceso a Datos (0486), semana real 15 del
calendario (11-17 enero, tras las vacaciones de Navidad). Sigue las convenciones de
estilo del README.md del repo.

Criterios de evaluación de RA5 que cubre este apartado (curriculum.md):
- a) Ventajas e inconvenientes de utilizar bases de datos documentales nativas.
- d) Se han añadido y eliminado colecciones de la base de datos.

Contenido central: comparación razonada entre modelo relacional (PostgreSQL, ya
trabajado en los Temas 1-3) y modelo documental nativo (MongoDB, empezado en el apartado
anterior), usando GameVault como caso de estudio real de por qué conviven ambos en el
mismo proyecto.

ESTRUCTURA — teoría primero: antes del caso GameVault, cubre la parte general con
ejemplos genéricos en mongosh (comandos mostrados): cómo se crea una colección
(implícitamente al insertar el primer documento, o explícitamente con createCollection y
para qué sirve la forma explícita — validación, colecciones capadas), cómo se elimina
(dropCollection) y cómo se listan (show collections); y la comparación
relacional/documental como tabla de conceptos equivalentes (tabla↔colección,
fila↔documento, columna↔campo, JOIN↔documento embebido o referencia manual) con las
ventajas e inconvenientes de cada modelo razonados, no como lista memorística.

Apóyate en el proyecto GameVault (com.aleroig.gamevault) como ejemplo real:
- Explica por qué el catálogo (Videojuego/Estudio, con relaciones claras y necesidad de
  transacciones ACID fuertes entre entidades) vive en PostgreSQL, mientras que las
  reseñas (com/aleroig/gamevault/reviews/Review.java: documentos independientes, sin
  relaciones entre sí, con forma que podría variar con el tiempo) viven en MongoDB — esto
  es una decisión de arquitectura real documentada en
  docs/04-decisiones-arquitectura.md del proyecto (sección "Bases de datos
  dockerizadas"), cítala como ejemplo de decisión consciente, no de "usar NoSQL porque
  sí".
- Ventajas del modelo documental: esquema flexible por documento (dos Review no tienen
  por qué tener exactamente los mismos campos si el esquema evoluciona), escritura/lectura
  simple sin JOINs.
- Inconvenientes: la propia "integridad referencial manual" ya vista en el apartado
  anterior (videojuegoId sin clave foránea real) es el precio a pagar; no hay
  transacciones multi-documento tan robustas como las de PostgreSQL (aunque MongoDB
  moderno sí las soporta parcialmente, coméntalo como matiz, no como ausencia total).
- com/aleroig/gamevault/reviews/ReviewRepository.java, método
  `deleteByVideojuegoId(Long videojuegoId)`: úsalo como ejemplo de "eliminar documentos
  de una colección de forma masiva" — explica que esto es lo que en el proyecto real se
  invocaría si se quisiera limpiar las reseñas huérfanas detectadas en la Actividad 4.1
  cuando se borra un videojuego (aunque en el GameVault actual ese borrado en cascada NO
  está conectado automáticamente a la eliminación del videojuego — señálalo como una
  posible MEJORA que el alumnado podría plantearse, sin implementarla obligatoriamente
  aquí).
- Sobre "añadir y eliminar colecciones" en sí: en MongoDB una colección se crea
  implícitamente al guardar el primer documento (no hace falta una sentencia DDL como
  `CREATE TABLE`) y se elimina con `dropCollection` — contrástalo con el DDL de
  PostgreSQL (`ddl-auto` de Hibernate, visto en el Tema 1) para remarcar la diferencia de
  filosofía.

No entres todavía en modificación de documentos individuales (PUT) ni en el cierre de
RA5: eso es el siguiente apartado, modificacion-documentos.md.
```
