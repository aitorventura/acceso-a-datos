<a id="consultas-jpql-transacciones"></a>

# 🧩 8. Consultas JPQL y transacciones

Añades la tercera y última vía de consulta de Spring Data: para lo que ni un método derivado por nombre ni una Specification pueden expresar bien.

---

## 🔤 Qué es JPQL

**JPQL** (*Jakarta Persistence Query Language*) es el lenguaje de consultas de la propia especificación JPA — se parece a SQL, pero con una diferencia de fondo: opera sobre **entidades y sus propiedades**, no sobre tablas y columnas.

```sql
-- SQL: tablas y columnas
SELECT e.* FROM editorial e
JOIN libro l ON l.editorial_id = e.id
GROUP BY e.id;
```

```
-- JPQL: entidades y propiedades
SELECT e FROM Editorial e
JOIN e.libros l
GROUP BY e
```

Fíjate en las diferencias: `Editorial` con mayúscula inicial (es la clase Java, no la tabla `editorial`); `e.libros` navega la relación con un punto, tal como la declaraste en la entidad, en vez de un `JOIN ... ON` explícito sobre claves foráneas; no hay ninguna columna nombrada literalmente, solo propiedades del objeto.

---

## 🛤️ Las tres vías de consulta de Spring Data

Ya conoces dos; hoy llega la tercera:

| Vía | Cuándo usarla |
|---|---|
| **Naming de método** (`findByTitulo`, `findByEditorialId`) | Consultas simples, fijas, sin lógica adicional. |
| **Specifications** (apartado anterior) | Filtros dinámicos, combinables en tiempo de ejecución. |
| **`@Query` con JPQL** | Consultas complejas y fijas — típicamente, agregaciones. |

Una **agregación** (por si el repaso de SQL lo necesita) es una operación que resume varias filas en un único resultado: `COUNT` (cuántas), `SUM` (suma), `AVG` (media)... normalmente combinada con `GROUP BY`, que agrupa las filas antes de aplicar la agregación a cada grupo por separado.

---

## 📊 Un ejemplo completo: el ranking de editoriales

Siguiendo con la librería: imagina que quieres un ranking de editoriales ordenado por cuántos libros tiene cada una en el catálogo. Es una consulta fija (siempre la misma), pero con `JOIN`, agregación y ordenación por el resultado de esa agregación — el caso típico de `@Query`.

### La consulta con `@Query`

```java
public interface EditorialRepository extends JpaRepository<Editorial, Long> {

    @Query("SELECT e FROM Editorial e JOIN e.libros l GROUP BY e ORDER BY COUNT(l) DESC")
    List<Editorial> rankingPorNumeroDeLibros();
}
```

`@Query` sobre un método de repositorio te permite escribir tú mismo el JPQL cuando el naming de método o las Specifications se quedan cortos — aquí, porque necesitas un `JOIN` con agregación y ordenación por el resultado de esa agregación, algo que ninguna de las otras dos vías expresa con naturalidad.

### Cuando ni JPQL alcanza: `nativeQuery = true`

JPQL cubre la mayoría de casos, pero hay funcionalidad que directamente no existe en JPQL — por ejemplo, las **funciones de ventana** (*window functions*) de SQL, como `ROW_NUMBER()`. Imagina que quieres que el propio ranking de editoriales devuelva ya la posición numérica de cada una (1º, 2º, 3º...), en vez de calcularla tú en Java recorriendo la lista después. Eso es SQL de verdad, no JPQL:

```java
@Query(value = """
        SELECT e.*, ROW_NUMBER() OVER (ORDER BY COUNT(l.id) DESC) AS posicion
        FROM editorial e JOIN libro l ON l.editorial_id = e.id
        GROUP BY e.id
        """, nativeQuery = true)
List<Object[]> rankingConPosicion();
```

`nativeQuery = true` le dice a Spring Data que ese texto no es JPQL — es SQL tal cual lo entiende tu gestor, con sus tablas, columnas y funciones específicas (aquí, `ROW_NUMBER() OVER (...)`, que JPQL no soporta bajo ningún concepto). Tiene dos costes reales: pierdes la independencia de motor que sí tenía JPQL (esta consulta concreta solo funciona en gestores que soporten esa sintaxis), y el resultado deja de ser una lista de `Editorial` — pasa a ser `List<Object[]>` (cada fila, tal cual la devuelve la base de datos), porque el motor no sabe mapear una columna calculada como `posicion` a ningún campo de tu entidad. Recorrer ese `Object[]` fila a fila, extrayendo cada valor por posición, es el precio a pagar por pedirle a la base de datos algo que JPQL no puede expresar.

### Transacciones, desde el ángulo de las consultas

Ya conoces `@Transactional` desde el Tema 1. Aquí lo retomas centrado en las consultas:

```java
@Transactional(readOnly = true)
public List<EditorialRankingDTO> obtenerRanking() {
    return editorialRepository.rankingPorNumeroDeLibros()
            .stream()
            .map(e -> new EditorialRankingDTO(e.getNombre(), e.getLibros().size()))
            .toList();
}
```

`@Transactional(readOnly = true)` en operaciones de solo lectura no es solo un matiz estilístico: es una optimización real — le indica al framework y al gestor de base de datos que esta operación no va a modificar nada, lo que permite evitar bloqueos (*locks*) innecesarios que sí harían falta en una escritura. En operaciones de escritura, sigues usando `@Transactional` a secas, exactamente como en el `create()`/`update()` del Tema 1.

---

## ✅ Ideas clave

??? tip "Abrir resumen"

    - **JPQL** opera sobre entidades y propiedades Java, no sobre tablas y columnas SQL.
    - Las tres vías de consulta de Spring Data: naming de método (simple y fijo), Specifications (dinámico), `@Query`/JPQL (complejo y fijo, típicamente agregaciones).
    - `@Query(nativeQuery = true)` escribe SQL literal cuando JPQL no llega (funciones de ventana, por ejemplo) — a cambio, pierdes independencia de motor y el resultado ya no son entidades, sino `Object[]` por fila.
    - Una **agregación** (`COUNT`, `SUM`, `AVG`... con `GROUP BY`) resume varias filas en un resultado.
    - `@Transactional(readOnly = true)` en consultas es una optimización real, no solo un matiz — evita bloqueos innecesarios.
