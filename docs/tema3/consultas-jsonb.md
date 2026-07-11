<a id="consultas-jsonb"></a>

# 🧩 2. Consultas sobre columnas JSONB

Ya sabes guardar objetos estructurados en una columna JSONB. El siguiente paso: filtrar por su contenido — algo que una condición `WHERE` normal no sabe hacer, porque ya no compara una columna, sino una **clave dentro de un objeto**.

---

## 🔍 El problema: consultar dentro de un JSON

Con una columna normal, un `WHERE` compara el valor completo de esa columna. Con JSONB, lo que quieres comprobar no es la columna entera, sino si existe (o qué vale) una clave concreta dentro del objeto que esa columna contiene. PostgreSQL ofrece operadores y funciones específicas para esto:

```sql
-- ¿Existe la clave "ebook" de primer nivel en el JSON?
SELECT * FROM libro WHERE jsonb_exists(detallesedicion, 'ebook');
-- equivalente con el operador ?
SELECT * FROM libro WHERE detallesedicion ? 'ebook';

-- Extraer el valor de una clave
SELECT detallesedicion -> 'ebook' FROM libro;       -- como JSON
SELECT detallesedicion ->> 'ebook' FROM libro;      -- como texto
```

`jsonb_exists(columna, clave)` (o el operador `?`, equivalente) comprueba si una clave de primer nivel existe en el JSON — sin importar su valor. `->` extrae el valor de una clave manteniéndolo como JSON (útil si vas a seguir navegando dentro); `->>` lo extrae como texto plano.

Fíjate en que esto es SQL puro, directamente contra PostgreSQL — antes de verlo envuelto en Java, conviene tenerlo claro a este nivel.

---

## 📚 Un ejemplo completo: `disponibleEnFormato`

Siguiendo con la librería: un filtro opcional más en el buscador — "solo libros disponibles en ebook" (o en tapa dura, o en bolsillo). El dato vive dentro de la columna JSONB `detallesEdicion`, así que la Specification necesita algo más que un `like`.

### La Specification, con `jsonb_exists` desde Criteria API

```java
public static Specification<Libro> disponibleEnFormato(String formato) {
    if (formato == null || formato.isBlank()) {
        return Specification.unrestricted();
    }

    return (root, query, criteriaBuilder) ->
            criteriaBuilder.isTrue(
                    criteriaBuilder.function(
                            "jsonb_exists",
                            Boolean.class,
                            root.get("detallesEdicion"),
                            criteriaBuilder.literal(formato)
                    )
            );
}
```

Compárala con las Specifications "normales" que ya conoces del Tema 2 (`tituloContiene`, `precioMayorOIgualA`), que usan métodos directos de `criteriaBuilder` como `like` o `greaterThanOrEqualTo` — funcionan porque operan sobre una columna con su propio tipo simple. Aquí no existe un método directo de Criteria API para "existe esta clave en este JSON", porque es una función **específica del motor** (PostgreSQL), no parte del estándar JPA. Por eso hace falta `criteriaBuilder.function("jsonb_exists", ...)`: es la forma de invocar una función SQL nativa arbitraria desde Criteria API, pasándole el nombre de la función, el tipo de resultado esperado (`Boolean.class`), y sus argumentos (la columna, y un literal con la clave a buscar).

### Combinada con el resto, de forma transparente

```java
Specification<Libro> spec = Specification
        .where(LibroSpecifications.tituloContiene(filtro.titulo()))
        .and(LibroSpecifications.precioMayorOIgualA(filtro.precioMin()))
        .and(LibroSpecifications.disponibleEnFormato(filtro.formato()));
```

Desde el punto de vista de quien usa el repositorio, filtrar por JSONB o por una columna relacional corriente es exactamente lo mismo — una Specification más, encadenada con `.and(...)` como cualquier otra. Toda la complejidad de "esto necesita una función nativa" queda encapsulada dentro del método `disponibleEnFormato`.

---

## ✏️ Modificar objetos JSONB: reemplazo, no *merge*

Un `update()` que recibe un `Map` nuevo reemplaza el contenido completo de `detallesEdicion`, no combina el nuevo contenido con el anterior — lo comprobarás tú mismo en la Actividad 3.1.

```java
libro.setDetallesEdicion(dto.detallesEdicion()); // sustituye el Map entero
```

¿Por qué no existe, en Hibernate/JPA estándar, una forma directa de hacer un "merge parcial" del JSON sin traer el objeto completo primero? Porque, desde el punto de vista de Hibernate, `detallesEdicion` es un único valor de columna — igual que `titulo` o `precio`. JPA no sabe "mirar dentro" de ese valor para combinar solo una parte; trata el `Map` completo como una unidad atómica que se sustituye entera. Si quisieras un *merge* parcial de verdad, tendrías que cargar el objeto, modificar el `Map` en memoria en Java (añadiendo o quitando claves tú mismo) y luego guardar el resultado completo — la combinación ocurre en tu código, no en el ORM.

---

## 🔄 Transacciones: nada especial que gestionar

Merece la pena decirlo explícitamente para no generar expectativas de complejidad añadida — `@Transactional` funciona **exactamente igual** sobre una entidad con columnas JSONB que sobre cualquier otra entidad. No hay ningún matiz especial de transacciones que gestionar por tener una columna JSONB; sigue siendo la misma anotación, el mismo comportamiento, que ya conoces desde el Tema 1.

---

## ✅ Ideas clave

??? tip "Abrir resumen"

    - `jsonb_exists(columna, clave)` (o el operador `?`) comprueba si una clave existe en el JSON; `->`/`->>` extraen valores (como JSON o como texto).
    - Consultar el contenido de una columna JSONB desde Criteria API requiere `criteriaBuilder.function(...)`, porque son funciones específicas del motor, no parte del estándar JPA.
    - Combinada con `.and(...)`, una Specification sobre JSONB se usa exactamente igual que una sobre una columna normal — transparente para quien consume el repositorio.
    - Actualizar un campo JSONB **reemplaza** el objeto completo — JPA no sabe hacer *merge* parcial de un valor de columna; si lo necesitas, lo haces tú en memoria antes de guardar.
    - `@Transactional` no cambia en nada por tener columnas JSONB de por medio.
