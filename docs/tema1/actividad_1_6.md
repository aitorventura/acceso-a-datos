# 🧪 Actividad 1.6: Ranking con `@Query` JPQL

!!! warning "Descarga la plantilla"
    📄 [Plantilla 1.6 — Ranking con @Query JPQL](plantillas/Actividad_1_6_AD_Plantilla.docx){target="_blank" rel="noopener"}

!!! info "Práctica guiada"
    Hoy añades a tu GameVault un ranking de estudios por número de videojuegos publicados, usando `@Query` con JPQL.

## Qué vas a practicar

- Escribir una consulta JPQL con `@Query` sobre un repositorio.
- Filtrar con una consulta JPQL parametrizada, usando `@Param`.
- Diseñar un DTO de respuesta que no exponga más de lo necesario.
- Escribir una consulta con `@Query(nativeQuery = true)` para lo que JPQL no puede expresar.
- Modificar datos en bloque con `@Modifying`, en una transacción de escritura.

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
@Operation(summary = "Ranking de estudios por número de videojuegos publicados")
@ApiResponses({
        @ApiResponse(responseCode = "200", description = "Ranking completo, ordenado de más a menos videojuegos")
})
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

**Pregunta de comprensión**: ¿qué ventaja concreta aporta `readOnly = true` en este método, frente a dejarlo con `@Transactional` a secas? (Ya la has visto en la teoría de "Operaciones CRUD y gestión de transacciones" — enúnciala con tus propias palabras).

---

## Paso 4 — criterio de desempate

Ahora mismo, si dos estudios tienen exactamente el mismo número de videojuegos, su orden relativo en el ranking no está garantizado de forma predecible. Añade al `ORDER BY` de tu consulta JPQL un segundo criterio: nombre alfabético (`e.nombre ASC`) como desempate.

Para comprobarlo de verdad, crea (o ajusta) dos estudios con exactamente el mismo número de videojuegos y verifica que aparecen ordenados alfabéticamente entre sí, no en un orden arbitrario.

---

## Paso 5 — la posición exacta, con SQL nativo

Tu ranking ordena bien, pero no incluye la posición (1º, 2º, 3º...) como un dato explícito — para saberla, alguien tendría que contar el índice de cada elemento en la lista. Pídesela directamente a la base de datos con una función de ventana, igual que en el ejemplo de la teoría.

Añade a `EstudioRepository` un método `List<Object[]> rankingConPosicion()`, con `@Query(nativeQuery = true)`, que use `ROW_NUMBER() OVER (ORDER BY COUNT(v.id) DESC)` sobre las tablas `estudio`/`videojuego` (mismo patrón que el ejemplo de `Editorial`/`Libro` de la teoría, adaptado a tus tablas) para devolver, por cada estudio, su nombre, su número de videojuegos y su posición en el ranking.

**Pista**: el resultado será `List<Object[]>`, no `List<Estudio>` — vas a necesitar un DTO nuevo, y seleccionar esas mismas tres columnas explícitamente en tu `SELECT` (no `e.*`), para saber en qué posición del `Object[]` viene cada una:

```java
public record EstudioRankingConPosicionDTO(
        String nombreEstudio,
        int numeroDeVideojuegos,
        int posicion
) {}
```

Construir el service y el controller no es el objetivo de este paso —eso ya lo has practicado— así que aquí tienes los dos enteros. Solo te falta rellenar el mapeo de `Object[]` a DTO, según el orden de columnas que hayas elegido en tu propia consulta:

```java
// En EstudioService
@Transactional(readOnly = true)
public List<EstudioRankingConPosicionDTO> obtenerRankingConPosicion() {
    return estudioRepository.rankingConPosicion().stream()
            .map(fila -> new EstudioRankingConPosicionDTO(
                    /* rellena aquí el mapeo, según el orden de tu SELECT */
            ))
            .toList();
}
```

```java
// En EstudioController
@Operation(summary = "Ranking de estudios con su posición numérica")
@ApiResponses({
        @ApiResponse(responseCode = "200", description = "Ranking completo, con la posición de cada estudio ya calculada")
})
@GetMapping("/ranking-con-posicion")
public ResponseEntity<List<EstudioRankingConPosicionDTO>> getRankingConPosicion() {
    return ResponseEntity.ok(estudioService.obtenerRankingConPosicion());
}
```

**Captura**: la petición y la respuesta de `GET /api/v1/estudios/ranking-con-posicion` desde Swagger UI, con la posición ya calculada en cada fila.

**Pregunta**: ¿por qué esta consulta ya no es JPQL, aunque se parezca tanto? ¿Qué parte concreta de la sintaxis es la que JPQL no podría expresar?

---

## Paso 6 — busca por nombre de estudio, con `@Param`

Añade a `VideojuegoRepository` un método `List<Videojuego> buscarPorNombreDeEstudio(String nombreEstudio)`, que busque todos los videojuegos publicados por el estudio cuyo nombre exacto le pases (no su `id`) — mismo patrón que el ejemplo de `Libro`/`Editorial` de la teoría ("JPQL con parámetros"), adaptado a tus entidades.

**Pista**: la consulta navega la relación `v.estudio.nombre`, y el parámetro se conecta con `@Param`.

Construir el service y el controller no es el objetivo de este paso, así que aquí tienes los dos enteros:

```java
// En VideojuegoService
@Transactional(readOnly = true)
public List<VideojuegoResponseDTO> buscarPorEstudio(String nombreEstudio) {
    return videojuegoRepository.buscarPorNombreDeEstudio(nombreEstudio).stream()
            .map(this::mapToDTO)
            .toList();
}
```

```java
// En VideojuegoController
@Operation(summary = "Listar los videojuegos publicados por un estudio, buscando por su nombre exacto")
@ApiResponses({
        @ApiResponse(responseCode = "200", description = "Videojuegos del estudio indicado (vacía si no tiene ninguno, o si el nombre no existe)")
})
@GetMapping("/por-nombre-de-estudio/{nombre}")
public ResponseEntity<List<VideojuegoResponseDTO>> getPorEstudio(@PathVariable String nombre) {
    return ResponseEntity.ok(videojuegoService.buscarPorEstudio(nombre));
}
```

**Captura**: la petición y la respuesta de `GET /api/v1/videojuegos/por-nombre-de-estudio/{nombre}` desde Swagger UI, con el nombre exacto de uno de tus estudios sembrados.

**Pregunta**: si escribieras mal el nombre del parámetro en `@Param(...)` —que no coincidiera con el `:parámetro` del JPQL—, ¿en qué momento te enterarías del error: al arrancar la aplicación, o al ejecutar la consulta? Relaciónalo con lo que dice la teoría sobre cuándo valida Spring la sintaxis de un `@Query`.

---

## Paso 7 — un descuento en bloque, con `@Modifying`

Añade a `VideojuegoRepository` un método `int aplicarDescuento(Long estudioId)`, que aplique un descuento del 10% a todos los videojuegos del estudio cuyo `id` le pases, en una sola consulta — mismo patrón que el ejemplo de `Libro`/`Editorial` de la teoría ("Transacciones al escribir con JPQL").

**Pista**: necesitas `@Modifying` además de `@Query`, una transacción de escritura (no `readOnly`), y el método devuelve un `int`, no una lista.

Construir el service y el controller no es el objetivo de este paso, así que aquí tienes los dos enteros:

```java
// En VideojuegoService
@Transactional
public int aplicarDescuento(Long estudioId) {
    return videojuegoRepository.aplicarDescuento(estudioId);
}
```

```java
// En VideojuegoController
@Operation(summary = "Aplicar un descuento del 10% a todos los videojuegos de un estudio")
@ApiResponses({
        @ApiResponse(responseCode = "200", description = "Descuento aplicado; el cuerpo indica cuántos videojuegos se han visto afectados")
})
@PostMapping("/descuento/{estudioId}")
public ResponseEntity<Map<String, Integer>> aplicarDescuento(@PathVariable Long estudioId) {
    int actualizados = videojuegoService.aplicarDescuento(estudioId);
    return ResponseEntity.ok(Map.of("videojuegosActualizados", actualizados));
}
```

**Captura**: la petición y la respuesta de `POST /api/v1/videojuegos/descuento/{estudioId}` desde Swagger UI, sobre un estudio con varios videojuegos sembrados.

Comprueba, con una consulta normal después, que los precios han cambiado de verdad.

**Pregunta**: ¿por qué este método no puede devolver `List<Videojuego>` con los videojuegos ya actualizados, aunque quisieras mostrárselos al usuario justo después de aplicar el descuento?

---

## Síntesis propia

Con tus propias palabras (2-3 frases, no copiadas del enunciado ni de la teoría), explica cómo esta actividad, junto con las dos anteriores del tema (mapeo con anotaciones, Specifications dinámicas), te ha permitido consultar tu catálogo de formas muy distintas. ¿Qué tipo de consulta usarías para cada una de las tres situaciones siguientes, y por qué: "buscar un videojuego por su id exacto", "listar videojuegos filtrando por hasta cuatro criterios opcionales", "calcular cuántos videojuegos ha publicado cada estudio"?

---

## ✅ Cierre

En el Tema 3 das el salto a bases de datos objeto-relacionales: vas a construir `disponibleEnPlataforma`, una Specification que usa `jsonb_exists` — ahora la vas a entender de verdad, trabajando con la columna JSONB que la hace posible.
