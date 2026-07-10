<a id="consultas-jpql-transacciones"></a>

# 🧩 3. Consultas JPQL y transacciones

Añades la tercera y última vía de consulta de Spring Data: para lo que ni un método derivado por nombre ni una Specification pueden expresar bien.

---

## 🔤 Qué es JPQL

**JPQL** (*Jakarta Persistence Query Language*) es el lenguaje de consultas de la propia especificación JPA — se parece a SQL, pero con una diferencia de fondo: opera sobre **entidades y sus propiedades**, no sobre tablas y columnas.

```sql
-- SQL: tablas y columnas
SELECT e.* FROM estudio e
JOIN videojuego v ON v.estudio_id = e.id
GROUP BY e.id;
```

```
-- JPQL: entidades y propiedades
SELECT e FROM Estudio e
JOIN e.videojuegos v
GROUP BY e
```

Fíjate en las diferencias: `Estudio` con mayúscula inicial (es la clase Java, no la tabla `estudio`); `e.videojuegos` navega la relación con un punto, tal como la declaraste en la entidad, en vez de un `JOIN ... ON` explícito sobre claves foráneas; no hay ninguna columna nombrada literalmente, solo propiedades del objeto.

---

## 🛤️ Las tres vías de consulta de Spring Data

Ya conoces dos; hoy llega la tercera:

| Vía | Cuándo usarla |
|---|---|
| **Naming de método** (`findByTitulo`, `findByEstudioId`) | Consultas simples, fijas, sin lógica adicional. |
| **Specifications** (Tema 2, apartado 2) | Filtros dinámicos, combinables en tiempo de ejecución. |
| **`@Query` con JPQL** | Consultas complejas y fijas — típicamente, agregaciones. |

Una **agregación** (por si el repaso de SQL lo necesita) es una operación que resume varias filas en un único resultado: `COUNT` (cuántas), `SUM` (suma), `AVG` (media)... normalmente combinada con `GROUP BY`, que agrupa las filas antes de aplicar la agregación a cada grupo por separado.

---

## 🎮 Aterrizaje en GameVault: un ranking que no existe todavía

Ahora mismo, tu propio `EstudioRepository.java` es un `JpaRepository` plano, sin un solo método propio. No hay ningún ranking de estudios implementado — es la mejora que vas a construir tú, siguiendo el estilo del resto del proyecto (mismo paquete `catalogo`, mismas convenciones con Lombok/`@RequiredArgsConstructor`).

### La consulta con `@Query`

```java
public interface EstudioRepository extends JpaRepository<Estudio, Long> {

    @Query("SELECT e FROM Estudio e JOIN e.videojuegos v GROUP BY e ORDER BY COUNT(v) DESC")
    List<Estudio> rankingPorNumeroDeVideojuegos();
}
```

`@Query` sobre un método de repositorio te permite escribir tú mismo el JPQL cuando el naming de método o las Specifications se quedan cortos — aquí, porque necesitas un `JOIN` con agregación y ordenación por el resultado de esa agregación, algo que ninguna de las otras dos vías expresa con naturalidad.

### Transacciones, desde el ángulo de las consultas

Ya conoces `@Transactional` desde el Tema 1. Aquí lo retomas centrado en las consultas:

```java
@Transactional(readOnly = true)
public List<EstudioRankingDTO> obtenerRanking() {
    return estudioRepository.rankingPorNumeroDeVideojuegos()
            .stream()
            .map(e -> new EstudioRankingDTO(e.getNombre(), e.getVideojuegos().size()))
            .toList();
}
```

`@Transactional(readOnly = true)` en operaciones de solo lectura no es solo un matiz estilístico: es una optimización real — le indica al framework y al gestor de base de datos que esta operación no va a modificar nada, lo que permite evitar bloqueos (*locks*) innecesarios que sí harían falta en una escritura. En operaciones de escritura, sigues usando `@Transactional` a secas, exactamente como en `VideojuegoService.create()`/`update()` del Tema 1.

---

## 🧭 Recapitulación del tema

Con este apartado se completa el recorrido: instalación y configuración del ORM (Hibernate, incluido en `spring-boot-starter-data-jpa`) → mapeo con anotaciones (`@Entity`, `@Column`, relaciones) → persistencia con Specifications dinámicas y paginación → consultas JPQL para lo que las otras vías no cubren. El Tema 3 retoma exactamente donde este tema deja una pista suelta: `VideojuegoSpecifications.disponibleEnPlataforma`, mencionada de pasada en el apartado anterior, que trabaja sobre una columna JSONB — el tema completo de bases de datos objeto-relacionales.

---

## ✅ Ideas clave

??? tip "Abrir resumen"

    - **JPQL** opera sobre entidades y propiedades Java, no sobre tablas y columnas SQL.
    - Las tres vías de consulta de Spring Data: naming de método (simple y fijo), Specifications (dinámico), `@Query`/JPQL (complejo y fijo, típicamente agregaciones).
    - Una **agregación** (`COUNT`, `SUM`, `AVG`... con `GROUP BY`) resume varias filas en un resultado.
    - `@Transactional(readOnly = true)` en consultas es una optimización real, no solo un matiz — evita bloqueos innecesarios.
