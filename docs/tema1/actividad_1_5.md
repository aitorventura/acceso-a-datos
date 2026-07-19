# 🧪 Actividad 1.5: Consultas dinámicas con Specifications

!!! info "Práctica guiada"
    Vas a construir el sistema de filtros dinámicos del listado de videojuegos: varias Specifications combinables, un DTO de filtro y paginación real.

## Qué vas a practicar

- Habilitar Specifications en un repositorio.
- Construir condiciones de consulta combinables, con el patrón `unrestricted()` para filtros ausentes.
- Combinar varias Specifications y aplicar paginación en el service.

---

## Requisitos previos

Tu CRUD de `Videojuego` del Tema 1, con varios videojuegos de distinto precio y estudio ya creados.

---

## Paso 1 — Preparación: `JpaSpecificationExecutor` y la clase de Specifications

Modifica tu `VideojuegoRepository`:

```java
public interface VideojuegoRepository extends
        JpaRepository<Videojuego, Long>,
        JpaSpecificationExecutor<Videojuego> {
}
```

Crea la clase `VideojuegoSpecifications` en el paquete `catalogo`:

```java
package com.tunombre.gamevault.catalogo;

import org.springframework.data.jpa.domain.Specification;
import java.math.BigDecimal;

public final class VideojuegoSpecifications {

    private VideojuegoSpecifications() { }

    // los métodos van aquí
}
```

El constructor privado y la clase `final` son deliberados: esta clase es una simple colección de métodos estáticos, no tiene sentido instanciarla.

---

## Paso 2 — Primera Specification, guiada al completo

```java
public static Specification<Videojuego> tituloContiene(String titulo) {
    if (titulo == null || titulo.isBlank()) {
        return Specification.unrestricted();
    }

    return (root, query, criteriaBuilder) ->
            criteriaBuilder.like(
                    criteriaBuilder.lower(root.get("titulo")),
                    "%" + titulo.toLowerCase() + "%"
            );
}
```

Primero, el caso "sin filtro": si `titulo` es `null` o está vacío, no queremos añadir ninguna condición — `Specification.unrestricted()` es justo eso. Si hay un valor, devolvemos una función `(root, query, criteriaBuilder) -> ...` que construye la condición: `criteriaBuilder.lower(...)` pasa el campo a minúsculas antes de comparar (para que la búsqueda no distinga mayúsculas), y `like(..., "%valor%")` busca coincidencias parciales, no exactas.

---

## Paso 3 — Segunda Specification: comparación numérica

```java
public static Specification<Videojuego> precioMayorOIgualA(BigDecimal precioMin) {
    if (precioMin == null) {
        return Specification.unrestricted();
    }

    return (root, query, criteriaBuilder) ->
            criteriaBuilder.greaterThanOrEqualTo(root.get("precio"), precioMin);
}
```

Mismo patrón, nuevo matiz: `greaterThanOrEqualTo` para comparar un `BigDecimal`, en vez de `like` para texto.

---

## Mini-retos — repite el patrón dos veces más

Sin más código dado, escribe tú:

1. `precioMenorOIgualA(BigDecimal precioMax)`: filtra videojuegos con precio menor o igual al indicado. Mismo patrón que `precioMayorOIgualA`, con el operador contrario.
2. `perteneceAlEstudio(Long estudioId)`: filtra videojuegos que pertenecen a un estudio concreto (compara `root.get("estudio").get("id")` con `estudioId` usando `criteriaBuilder.equal`).

No incluyas todavía ninguna Specification sobre `detallesPlataforma` (JSONB) — eso es contenido del Tema 3.

---

## Paso 4 — Combinar en el service, con paginación

Crea el DTO de filtro:

```java
public record VideojuegoFiltroDTO(
        String titulo,
        BigDecimal precioMin,
        BigDecimal precioMax,
        Long estudioId
) {}
```

Y en `VideojuegoService`:

```java
@Transactional(readOnly = true)
public Page<VideojuegoResponseDTO> findAllPaginated(VideojuegoFiltroDTO filtro, Pageable pageable) {
    Specification<Videojuego> spec = Specification
            .where(VideojuegoSpecifications.tituloContiene(filtro.titulo()))
            .and(VideojuegoSpecifications.precioMayorOIgualA(filtro.precioMin()))
            .and(VideojuegoSpecifications.precioMenorOIgualA(filtro.precioMax()))
            .and(VideojuegoSpecifications.perteneceAlEstudio(filtro.estudioId()));

    return videojuegoRepository.findAll(spec, pageable)
            .map(this::mapToDTO);
}
```

En el controller:

```java
@GetMapping
public ResponseEntity<Page<VideojuegoResponseDTO>> getAll(
        @ModelAttribute VideojuegoFiltroDTO filtro,
        @PageableDefault(size = 5) Pageable pageable
) {
    return ResponseEntity.ok(videojuegoService.findAllPaginated(filtro, pageable));
}
```

`@ModelAttribute` mapea automáticamente los parámetros de query string (`?titulo=...&precioMin=...`) a los campos del DTO; `@PageableDefault(size = 5)` fija un tamaño de página por defecto si el cliente no especifica ninguno.

---

!!! warning "Esto rompe el test MockMvc que ya tienes de PSP"
    `VideojuegoControllerTest` (Programación de Servicios y Procesos, Actividad 1.3) esperaba una `List<VideojuegoResponseDTO>` y comprobaba `jsonPath("$[0].titulo")`. Ahora que `getAll()` devuelve una `Page<VideojuegoResponseDTO>`, ese `jsonPath` deja de tener sentido — el array ya no está en la raíz del JSON, sino dentro de `content`. Actualiza ese test: cambia el mock a `when(videojuegoService.findAllPaginated(any(), any())).thenReturn(...)` y el `jsonPath` a `"$.content[0].titulo"`.

## Paso 5 — Verificación con peticiones reales

```bash
# Sin filtros: debe devolver todo (paginado)
curl "http://localhost:8080/api/v1/videojuegos"

# Un solo filtro
curl "http://localhost:8080/api/v1/videojuegos?titulo=hades"

# Varios filtros combinados
curl "http://localhost:8080/api/v1/videojuegos?precioMin=10&precioMax=30&estudioId=1"
```

**Comprueba**, para cada caso, que el resultado tiene sentido: la petición sin filtros debe devolver todo el catálogo (paginado, no un error ni una lista vacía); la que combina varios filtros debe devolver solo lo que cumple **todas** las condiciones a la vez.

---

## Pregunta final

¿Por qué conviene que cada método de `VideojuegoSpecifications` devuelva `Specification.unrestricted()` cuando su filtro es `null`, en vez de resolverlo con un `if (filtro != null)` fuera del método, antes de encadenar los `.and(...)` en el service? Piensa en qué pasaría con el código del Paso 4 si tuvieras que añadir ese `if` para cada uno de los cuatro filtros.

---

## ✅ Cierre

Tu listado de videojuegos ya soporta filtros dinámicos y paginación real, con SQL generado automáticamente según qué combinación de filtros llegue. En la próxima actividad trabajas con `@Query` JPQL — para las consultas que ni las Specifications ni los métodos derivados por nombre pueden expresar bien, como una agregación.
