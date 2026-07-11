<a id="colecciones-documentales"></a>

# 🧩 2. Colecciones documentales: relacional vs. documental

Ya conectaste con MongoDB y construiste tu primer repositorio documental. Este apartado da un paso atrás para comparar con criterio los dos modelos que has usado en el curso, y cierra el hueco práctico que dejaste abierto: crear y eliminar colecciones explícitamente.

---

## 🆚 Relacional vs. documental, concepto a concepto

| Relacional (PostgreSQL) | Documental (MongoDB) |
|---|---|
| Tabla | Colección |
| Fila | Documento |
| Columna | Campo |
| `JOIN` | Documento embebido, o referencia manual (sin integridad automática) |

Ventajas del modelo documental: **esquema flexible por documento** (dos `Resena` no tienen por qué compartir exactamente los mismos campos si el esquema evoluciona con el tiempo, sin necesitar ninguna migración) y **escritura/lectura simple**, sin `JOIN` que resolver.

Inconvenientes: la **integridad referencial manual** que ya viste en el apartado anterior es el precio a pagar — nadie garantiza automáticamente que `libroId` apunte a algo real. Tampoco hay transacciones multi-documento tan robustas como las de PostgreSQL (MongoDB moderno sí las soporta parcialmente, pero es un matiz, no la ausencia total que tenía en sus primeras versiones).

---

## 🛠️ Colecciones: crear, eliminar, listar

En `mongosh`:

```javascript
// Se crea implícitamente al insertar el primer documento
db.resena.insertOne({ libroId: 1, autor: "ana", puntuacion: 8 })

// O explícitamente, cuando necesitas opciones concretas (validación, colecciones capadas)
db.createCollection("resena")

// Eliminar
db.resena.drop()

// Listar
show collections
```

La forma **explícita** (`createCollection`) sirve para cuando necesitas configurar algo desde el principio — por ejemplo, reglas de validación de esquema, o una colección "capada" (de tamaño fijo, que sobrescribe los documentos más antiguos). Para el caso normal, no hace falta: MongoDB crea la colección sola en cuanto guardas el primer documento.

!!! tip "Contraste con el DDL de PostgreSQL"
    En PostgreSQL, necesitas una sentencia `CREATE TABLE` explícita (o `ddl-auto` de Hibernate haciéndolo por ti, como viste en el Tema 1) antes de poder insertar nada. En MongoDB, la colección aparece sola al primer `insert` — es la misma diferencia de filosofía que ya viste entre esquema fijo y esquema flexible, ahora aplicada a la propia existencia de la colección.

---

## 📚 Por qué dos motores conviven

En el ejemplo de la librería, PostgreSQL guarda el catálogo (`Libro`/`Editorial`: relaciones claras, necesidad de transacciones ACID fuertes entre entidades relacionadas) y MongoDB guarda las reseñas (`Resena`: documentos independientes entre sí, sin relaciones que mantener, con forma que podría evolucionar). Esta es una **decisión de arquitectura deliberada**, no "usar NoSQL porque sí": cada motor se elige por lo que sus garantías resuelven mejor en cada parte del dominio.

### Eliminar documentos de forma masiva

```java
public interface ResenaRepository extends MongoRepository<Resena, String> {
    long deleteByLibroId(Long libroId);
}
```

`deleteByLibroId` es el ejemplo de "eliminar documentos de una colección de forma masiva", generado igual que cualquier otro método por naming — sin escribir ninguna query. Es exactamente el método que resuelve el problema de las **reseñas huérfanas** que detectaste en la Actividad 4.1: si se borra un libro cuyas reseñas siguen en Mongo, este método las limpia de golpe.

---

## ✅ Ideas clave

??? tip "Abrir resumen"

    - Tabla↔colección, fila↔documento, columna↔campo, `JOIN`↔documento embebido o referencia manual.
    - El modelo documental gana en flexibilidad de esquema y simplicidad de lectura/escritura; pierde integridad referencial automática y transacciones multi-documento tan robustas como en relacional.
    - Una colección se crea implícitamente al primer `insert`, o explícitamente con `createCollection` (para validación o colecciones capadas); se elimina con `drop()`.
    - Usar dos motores a la vez es una decisión de arquitectura deliberada: PostgreSQL para relaciones fuertes, MongoDB para documentos independientes.
    - `deleteByLibroId` elimina documentos en bloque por naming de método — es la pieza que resuelve las reseñas huérfanas.
