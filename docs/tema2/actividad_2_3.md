# 🧪 Actividad 2.3: Ranking con `@Query` JPQL

!!! info "Práctica guiada"
    Hoy añades a tu GameVault un ranking de estudios por número de videojuegos publicados, usando `@Query` con JPQL.

## Qué vas a practicar

- Escribir una consulta JPQL con `@Query` sobre un repositorio.
- Diseñar un DTO de respuesta que no exponga más de lo necesario.
- Aplicar `@Transactional(readOnly = true)` correctamente.

---

## Requisitos previos

Varios estudios con distinto número de videojuegos asociados (créalos si hace falta, para que el ranking tenga sentido al comprobarlo).

---

## Paso 1 — La consulta JPQL, guiada al completo

En `EstudioRepository`:

```java
public interface EstudioRepository extends JpaRepository<Estudio, Long> {

    @Query("SELECT e FROM Estudio e JOIN e.videojuegos v GROUP BY e ORDER BY COUNT(v) DESC")
    List<Estudio> rankingPorNumeroDeVideojuegos();
}
```

Compara esta consulta con su equivalente en SQL nativo:

```sql
SELECT e.* FROM estudio e
JOIN videojuego v ON v.estudio_id = e.id
GROUP BY e.id
ORDER BY COUNT(v.id) DESC;
```

`JOIN e.videojuegos v` navega la relación `@OneToMany` que ya tienes declarada en `Estudio` — no hace falta que escribas la condición del `JOIN` explícitamente (`ON v.estudio_id = e.id`), porque JPQL ya conoce esa relación por el propio mapeo de tu entidad. `GROUP BY e` agrupa por estudio completo (no por una columna suelta, como harías en SQL), y `ORDER BY COUNT(v) DESC` ordena por el tamaño de cada grupo, de mayor a menor.

---

## Paso 2 — El DTO de respuesta

Crea un DTO específico para el ranking — no reutilices `EstudioResponseDTO` ni devuelvas la entidad `Estudio` completa:

```java
public record EstudioRankingDTO(
        String nombreEstudio,
        int numeroDeVideojuegos
) {}
```

**Pregunta**: ¿por qué no tiene sentido devolver, dentro de este DTO, la lista completa de objetos `Videojuego` de cada estudio? Relaciona tu respuesta con lo que ya sabes sobre por qué GameVault nunca expone directamente sus entidades JPA (Tema 1).

---

## Paso 3 — El service y el endpoint

```java
// En EstudioService
@Transactional(readOnly = true)
public List<EstudioRankingDTO> obtenerRanking() {
    return estudioRepository.rankingPorNumeroDeVideojuegos()
            .stream()
            .map(e -> new EstudioRankingDTO(e.getNombre(), e.getVideojuegos().size()))
            .toList();
}
```

```java
// En EstudioController
@GetMapping("/ranking")
public ResponseEntity<List<EstudioRankingDTO>> getRanking() {
    return ResponseEntity.ok(estudioService.obtenerRanking());
}
```

Prueba:

```bash
curl http://localhost:8080/api/v1/estudios/ranking
```

**Comprueba**: que el orden de los estudios en la respuesta coincide con el número real de videojuegos que tiene cada uno en tu base de datos.

**Pregunta de comprensión**: ¿qué ventaja concreta aporta `readOnly = true` en este método, frente a dejarlo con `@Transactional` a secas? (Ya la viste en la teoría — enúnciala con tus propias palabras).

---

## Mini-reto — criterio de desempate

Ahora mismo, si dos estudios tienen exactamente el mismo número de videojuegos, su orden relativo en el ranking no está garantizado de forma predecible. Añade al `ORDER BY` de tu consulta JPQL un segundo criterio: nombre alfabético (`e.nombre ASC`) como desempate.

Para comprobarlo de verdad, crea (o ajusta) dos estudios con exactamente el mismo número de videojuegos y verifica que aparecen ordenados alfabéticamente entre sí, no en un orden arbitrario.

---

## Síntesis propia

Con tus propias palabras (2-3 frases, no copiadas del enunciado ni de la teoría), explica cómo esta actividad, junto con las dos anteriores del tema (mapeo con anotaciones, Specifications dinámicas), te ha permitido consultar tu catálogo de formas muy distintas. ¿Qué tipo de consulta usarías para cada una de las tres situaciones siguientes, y por qué: "buscar un videojuego por su id exacto", "listar videojuegos filtrando por hasta cuatro criterios opcionales", "calcular cuántos videojuegos ha publicado cada estudio"?

---

## ✅ Cierre

En el Tema 3 das el salto a bases de datos objeto-relacionales: vas a construir `disponibleEnPlataforma`, una Specification que usa `jsonb_exists` — ahora la vas a entender de verdad, trabajando con la columna JSONB que la hace posible.
