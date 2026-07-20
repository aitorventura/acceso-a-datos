# 🧪 Actividad 1.5: Consultas dinámicas con Specifications

!!! warning "Descarga la plantilla"
    📄 [Plantilla 1.5 — Consultas dinámicas con Specifications](plantillas/Actividad_1_5_AD_Plantilla.docx){target="_blank" rel="noopener"}

!!! info "Práctica guiada"
    Vas a construir el sistema de filtros dinámicos del listado de videojuegos: varias Specifications combinables, un DTO de filtro y paginación real.

## Qué vas a practicar

- Sembrar datos de prueba de forma automática y repetible con `data.sql`.
- Habilitar Specifications en un repositorio.
- Construir una condición de consulta combinable, y replicar el patrón tú mismo para el resto de filtros.
- Combinar varias Specifications y aplicar paginación en el service.
- Probar un endpoint con parámetros opcionales, tanto por `curl` como desde Swagger UI.

---

## Requisitos previos

Tu CRUD de `Videojuego` del Tema 1.

---

## Paso 0 — Mismos datos para todos, sembrados desde un fichero `data.sql`

Con los 3-4 videojuegos que tengas sueltos de actividades anteriores, la paginación es invisible: caben todos en una sola página y `@PageableDefault(size = 5)` nunca llega a mostrar una segunda. En vez de sembrarlos a mano por `curl` o `psql`, vas a usar la vía automática que ya conoces de la teoría: un fichero `data.sql`.

Crea `src/main/resources/data.sql`:

```sql
-- data.sql se ejecuta en cada arranque: TRUNCATE con RESTART IDENTITY borra
-- las filas Y reinicia el contador de id a 1 — un DELETE normal no reinicia
-- ese contador, así que los estudio_id de más abajo (1-5) podrían no existir
-- de verdad si ya habías creado algún estudio antes, y el INSERT siguiente
-- fallaría por clave foránea
TRUNCATE TABLE videojuego, estudio RESTART IDENTITY CASCADE;

INSERT INTO estudio (nombre, pais) VALUES
    ('Supergiant Games', 'Estados Unidos'),
    ('Team Cherry', 'Australia'),
    ('Motion Twin', 'Francia'),
    ('Toby Fox', 'Estados Unidos'),
    ('Klei Entertainment', 'Canadá');

INSERT INTO videojuego (titulo, precio, fecha_lanzamiento, estudio_id) VALUES
    ('Hades', 24.99, '2020-09-17', 1),
    ('Pyre', 19.99, '2017-07-25', 1),
    ('Transistor', 19.99, '2014-05-20', 1),
    ('Bastion', 14.99, '2011-07-20', 1),
    ('Hades II', 29.99, '2024-05-06', 1),
    ('Hollow Knight', 14.99, '2017-02-24', 2),
    ('Hollow Knight: Silksong', 19.99, '2025-09-04', 2),
    ('Dead Cells', 24.99, '2018-08-07', 3),
    ('Undertale', 9.99, '2015-09-15', 4),
    ('Mark of the Ninja', 14.99, '2012-09-07', 5),
    ('Don''t Starve', 14.99, '2013-04-23', 5),
    ('Oxygen Not Included', 24.99, '2019-07-30', 5);
```

Y en `application-dev.yaml`, dos propiedades — sin ellas, este fichero no llega a ejecutarse nunca, o falla al arrancar:

```yaml
spring:
  sql:
    init:
      mode: always
  jpa:
    defer-datasource-initialization: true
```

| Propiedad | Por qué hace falta |
|---|---|
| `spring.sql.init.mode: always` | Por defecto, Spring Boot solo ejecuta `data.sql` automáticamente contra bases de datos **embebidas** (H2). Tu PostgreSQL vive en un contenedor Docker aparte — sin esta propiedad, `data.sql` se ignoraría en silencio, sin ningún error. |
| `spring.jpa.defer-datasource-initialization: true` | Por defecto, Spring Boot ejecuta `data.sql` **antes** de que Hibernate cree las tablas con `ddl-auto`. Sin esta propiedad, el `INSERT` fallaría al arrancar: intentaría insertar en tablas que todavía no existen. Esta propiedad invierte el orden — primero el esquema, luego los datos. |

Reinicia la aplicación y comprueba con `GET /api/v1/videojuegos` (o `SELECT count(*) FROM videojuego;` por `psql`) que tienes 12 filas — suficientes para que, con el tamaño de página por defecto (5, en el Paso 3), salgan tres páginas distintas (5 + 5 + 2). Reinicia una segunda vez y comprueba que sigues teniendo exactamente 12, no 24: el `TRUNCATE` inicial es lo que evita que se dupliquen en cada arranque, además de garantizar que los `estudio_id` del `INSERT` de `videojuego` (1 a 5) siempre coinciden con los ids reales.

**Captura**: el resultado de `SELECT count(*) FROM videojuego;` mostrando 12, tras el segundo reinicio.

!!! warning "Cómo dejar de sembrar datos cuando ya no lo necesites"
    `data.sql` va a seguir ejecutándose en **cada** arranque mientras exista el fichero y `spring.sql.init.mode` esté en `always` — incluso el día que ya tengas tus propios datos de prueba metidos a mano y no quieras perderlos al reiniciar. Para desactivarlo no hace falta tocar código, basta una de estas dos cosas:

    - **Borra (o renombra) `data.sql`** — sin el fichero, no hay nada que ejecutar.
    - **Cambia `spring.sql.init.mode` a `never`** (o quita la propiedad entera, que es su valor por defecto) — conserva el fichero por si quieres reactivarlo más adelante, pero deja de ejecutarse mientras tanto.

    En los dos casos tu base de datos existente se queda tal cual está — `defer-datasource-initialization` deja de tener ningún efecto si `data.sql` no llega a ejecutarse.

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

## Paso 2 — Tu primera Specification, guiada al completo

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

## Mini-retos — repite el patrón tres veces más

Sin más código dado, escribe tú las otras tres Specifications de `VideojuegoSpecifications`, siguiendo exactamente el mismo patrón que `tituloContiene`: primero el caso "sin filtro" con `Specification.unrestricted()`, luego la condición real.

1. `precioMayorOIgualA(BigDecimal precioMin)`: filtra videojuegos con precio mayor o igual al indicado (`criteriaBuilder.greaterThanOrEqualTo`).
2. `precioMenorOIgualA(BigDecimal precioMax)`: filtra videojuegos con precio menor o igual al indicado. Mismo patrón que el anterior, con el operador contrario.
3. `perteneceAlEstudio(Long estudioId)`: filtra videojuegos que pertenecen a un estudio concreto (compara `root.get("estudio").get("id")` con `estudioId` usando `criteriaBuilder.equal`).

---

## Paso 3 — Combinar en el service, con paginación

Crea el DTO de filtro:

```java
public record VideojuegoFiltroDTO(
        String titulo,
        BigDecimal precioMin,
        BigDecimal precioMax,
        Long estudioId
) {}
```

Ahora, sin código dado: en la teoría de este apartado ("Todo junto: Specifications + paginación en el mismo listado") tienes el `service` y el `controller` completos, combinando las Specifications con `Pageable` sobre `Libro`. Adapta ese mismo patrón a tu `VideojuegoService` y tu `VideojuegoController` — mismo nombre de método (`findAllPaginated`), misma firma (recibe `filtro` y `pageable`, devuelve `Page`), las cuatro Specifications de este tema encadenadas con `.and(...)`, pero sobre `Videojuego` en vez de `Libro`.

En el controller, sustituye directamente el `getAll()` que ya tienes de la Actividad 1.2 — no lo dejes conviviendo con el nuevo, bórralo. `@ModelAttribute` mapea automáticamente los parámetros de query string (`?titulo=...&precioMin=...`) a los campos del DTO; `@PageableDefault(size = 5)` fija un tamaño de página por defecto si el cliente no especifica ninguno (usa `5`, no el `20` de la teoría, para que las tres páginas del Paso 0 se noten con solo 12 videojuegos).

Con eso funcionando, quedan dos retoques antes de dar el paso por terminado:

1. **Limpia el `findAll()` que ya no usa nadie.** Con `getAll()` apuntando ahora a `findAllPaginated`, el `findAll()` de `VideojuegoService` que devolvía `List<VideojuegoResponseDTO>` (Actividad 1.2) se queda sin ningún sitio que lo llame. Bórralo — dejar métodos sin usar "por si acaso" es la clase de cosa que se acumula sin que nadie se dé cuenta.

2. **Actualiza el `@Operation`/`@ApiResponses` de `getAll()`.** La documentación OpenAPI que ya tenías sobre este endpoint (Programación de Servicios y Procesos, Actividad 1.2) describía una lista simple — con los filtros y la paginación nuevos, ese texto se queda desfasado:

    ```java
    @Operation(summary = "Listar videojuegos, con filtros opcionales y paginación")
    @ApiResponses({
            @ApiResponse(responseCode = "200", description = "Página de videojuegos que cumplen los filtros indicados (puede estar vacía)")
    })
    ```

!!! warning "Esto rompe el test MockMvc que ya tienes de PSP"
    `VideojuegoControllerTest` (Programación de Servicios y Procesos, Actividad 1.3) esperaba una `List<VideojuegoResponseDTO>` y comprobaba `jsonPath("$[0].titulo")`. Ahora que `getAll()` devuelve una `Page<VideojuegoResponseDTO>`, ese `jsonPath` deja de tener sentido — el array ya no está en la raíz del JSON, sino dentro de `content`. Así tenías el test:

    ```java
    @Test
    void getAll_DebeDevolverListaDeVideojuegos() throws Exception {
        var dto = new VideojuegoResponseDTO(1L, "Hades", new BigDecimal("24.99"), LocalDate.of(2020, 9, 17), "Supergiant Games");
        when(videojuegoService.findAll()).thenReturn(List.of(dto));

        mockMvc.perform(get("/api/v1/videojuegos"))
                .andExpect(status().isOk())
                .andExpect(jsonPath("$[0].titulo").value("Hades"));
    }
    ```

    Y así debe quedar, con el mock y el `jsonPath` adaptados a `Page` (añade `import org.springframework.data.domain.Page;` y `import org.springframework.data.domain.PageImpl;`, y usa `any()` de Mockito para los dos parámetros que ya no te interesa fijar en detalle):

    ```java
    @Test
    void getAll_DebeDevolverVideojuegosPaginados() throws Exception {
        var dto = new VideojuegoResponseDTO(1L, "Hades", new BigDecimal("24.99"), LocalDate.of(2020, 9, 17), "Supergiant Games");
        Page<VideojuegoResponseDTO> pagina = new PageImpl<>(List.of(dto));

        when(videojuegoService.findAllPaginated(any(), any())).thenReturn(pagina);

        mockMvc.perform(get("/api/v1/videojuegos"))
                .andExpect(status().isOk())
                .andExpect(jsonPath("$.content[0].titulo").value("Hades"));
    }
    ```

---

## Paso 4 — Verificación con peticiones reales

### Por `curl`

```bash
# Sin filtros, página 0 (la primera): con 12 videojuegos y tamaño 5, deben salir 5
curl "http://localhost:8080/api/v1/videojuegos"

# Sin filtros, página 2 (la tercera y última): deben salir los 2 restantes
curl "http://localhost:8080/api/v1/videojuegos?page=2"

# Un solo filtro
curl "http://localhost:8080/api/v1/videojuegos?titulo=hades"

# Varios filtros combinados
curl "http://localhost:8080/api/v1/videojuegos?precioMin=10&precioMax=30&estudioId=1"
```

**Comprueba**, para cada caso, que el resultado tiene sentido: la página 0 debe traer `content` con 5 elementos y `totalElements: 12`; la página 2, los 2 restantes; la que combina varios filtros debe devolver solo lo que cumple **todas** las condiciones a la vez.

**Captura**: la respuesta de la petición con varios filtros combinados (`precioMin`, `precioMax` y `estudioId` a la vez), mostrando que solo aparecen los videojuegos que cumplen las tres condiciones.

### Por Swagger UI

Abre `http://localhost:8080/swagger-ui/index.html`, busca el `GET /api/v1/videojuegos` y despliégalo. Al pulsar **Try it out**, Swagger no te muestra una única caja de texto para toda la URL — genera un campo de formulario **por cada parámetro por separado**: `titulo`, `precioMin`, `precioMax` y `estudioId` (los del DTO, gracias a `@ModelAttribute`), más `page`, `size` y `sort` (los del `Pageable`). Cada campo es opcional: déjalo vacío si no quieres aplicar ese filtro.

Rellena, por ejemplo, `precioMin` con `15` y `size` con `3`, deja el resto vacío, y pulsa **Execute**. Swagger construye la URL final por ti (podrás verla justo encima de la respuesta, algo como `?precioMin=15&size=3`) y te muestra el `Page` completo devuelto, con sus mismos campos (`content`, `totalElements`...) que ya conoces de la teoría.

**Captura**: el formulario de Swagger UI con los campos rellenos que has elegido, junto con la respuesta obtenida debajo.

---

## Pregunta final

¿Por qué conviene que cada método de `VideojuegoSpecifications` devuelva `Specification.unrestricted()` cuando su filtro es `null`, en vez de resolverlo con un `if (filtro != null)` fuera del método, antes de encadenar los `.and(...)` en el service? Piensa en qué pasaría con el código del Paso 3 si tuvieras que añadir ese `if` para cada uno de los cuatro filtros.

---

## ✅ Cierre

Tu listado de videojuegos ya soporta filtros dinámicos y paginación real, con SQL generado automáticamente según qué combinación de filtros llegue. En la próxima actividad trabajas con `@Query` JPQL — para las consultas que ni las Specifications ni los métodos derivados por nombre pueden expresar bien, como una agregación.

!!! note "¿Y el `GET` de `Estudio`?"
    Lo ideal en un proyecto real sería aplicar este mismo patrón (Specifications + paginación) a cualquier listado que pueda crecer sin límite — incluido `GET /api/v1/estudios`, que sigue devolviendo una `List` plana desde la Actividad 1.2. Esta actividad no lo pide: con pocos estudios en el catálogo, el problema que motiva este apartado (demasiadas filas, filtros combinables) todavía no se nota ahí. Si algún día tu catálogo de estudios creciera de verdad, sabrías exactamente qué aplicarle.
