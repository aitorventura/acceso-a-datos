<a id="persistencia-objetos-jsonb"></a>

# 🧩 1. Persistencia de objetos con JSONB

!!! warning "🚧 Contenido pendiente de desarrollo"
    Esta página todavía no tiene la teoría redactada. Usa el prompt de más abajo con
    `/improve-notes`, apoyándote en el proyecto **GameVault** adjunto, para generar el
    contenido definitivo.

---

## Prompt para `/improve-notes`

```text
Redacta el apartado de teoría "Persistencia de objetos con JSONB" del Tema 3 (RA4 - BD
objeto relacionales y orientadas a objetos) del módulo Acceso a Datos (0486), semana real
10 del calendario. Sigue las convenciones de estilo del README.md del repo.

Criterios de evaluación de RA4 que cubre este apartado (curriculum.md):
- a) Ventajas e inconvenientes de las bases de datos que almacenan objetos.
- b) Establecimiento y cierre de conexiones — en PostgreSQL con JSONB usamos el MISMO
  PostgreSQL y el mismo datasource ya configurado en el Tema 1: no hay una conexión
  nueva que abrir. Explica esto explícitamente (el propio calendario del curso marca
  este criterio como "no aplica — gestionado por el framework" y conviene que la teoría
  lo aclare en vez de ignorarlo).
- c) Persistencia de objetos simples.
- d) Persistencia de objetos estructurados.

ESTRUCTURA OBLIGATORIA — teoría primero, proyecto después.

PARTE 1 — Teoría general, desde cero:
- Qué es una base de datos objeto-relacional como categoría: un gestor relacional que
  además sabe almacenar y consultar OBJETOS (tipos compuestos, colecciones, documentos)
  dentro de sus tablas — el término del currículo, situado entre lo puramente relacional
  (Temas 1-2) y lo orientado a objetos puro (apartado 3 de este tema).
- Repaso breve de qué es JSON (el alumnado lo ha visto, pero no lo des por dominado):
  sintaxis mínima con un ejemplo (objetos, arrays, anidamiento) y por qué se ha
  convertido en el formato universal de intercambio.
- Objeto simple vs. objeto estructurado, con ejemplos genéricos: un valor escalar frente
  a un objeto con campos anidados y colecciones dentro.
- Qué ofrece PostgreSQL para esto: el tipo de columna jsonb (json binario, indexable y
  consultable) — una fila normal de una tabla normal puede llevar dentro un objeto
  entero con estructura variable, conviviendo con las columnas y claves foráneas de
  siempre. Diferencia json vs jsonb en una frase.

PARTE 2 — Aterrizaje en GameVault (com.aleroig.gamevault):
- com/aleroig/gamevault/catalogo/Videojuego.java: la propiedad
  `private Map<String, Object> detallesPlataforma;` anotada con
  `@JdbcTypeCode(SqlTypes.JSON)` y `@Column(columnDefinition = "jsonb")` — explica que
  esto le dice a Hibernate "mapea este campo Java (un Map) contra una columna jsonb de
  PostgreSQL", sin que el desarrollador tenga que serializar/deserializar JSON a mano.
- Diferencia "objeto simple" (un campo escalar como `titulo` o `precio`, persistencia
  "de toda la vida" ya vista en el Tema 2) de "objeto estructurado" (el propio
  `detallesPlataforma`, que puede contener claves anidadas como
  `{"steam": {"idApp": 123, "logros": 40}, "switch": {"idApp": "ABCD"}}`) — usa este
  ejemplo concreto para ilustrar qué es un objeto estructurado persistido tal cual, sin
  tener que crear una tabla nueva por cada plataforma.
- com/aleroig/gamevault/config/Jackson3HibernateMapper.java: menciónalo como la pieza que
  permite que Hibernate sepa serializar ese Map a JSON usando Jackson, sin entrar en
  detalles de implementación.

Explica también por qué el proyecto eligió JSONB en vez de crear una tabla
`detalle_plataforma` con más columnas o una tabla de relación 1:N: menciona el trade-off
(flexibilidad para añadir plataformas nuevas sin migración de esquema, a cambio de perder
las garantías de integridad referencial y de tipado estricto de una tabla normal).

No entres todavía en `jsonb_exists` ni en las consultas sobre el contenido del JSONB: eso
es el siguiente apartado, consultas-jsonb.md.
```
