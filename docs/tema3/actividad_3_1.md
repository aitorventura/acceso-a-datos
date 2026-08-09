# 🧪 Actividad 3.1: Reseñas de videojuegos en MongoDB

!!! info "Práctica guiada"
    Levantas MongoDB junto a tu PostgreSQL, y construyes el módulo `reviews` completo: entidad, repositorio, integridad referencial manual y los endpoints de consulta.

## Qué vas a practicar

- Añadir un segundo motor de base de datos al mismo proyecto.
- Crear una entidad documental con `@Document` y un repositorio `MongoRepository`.
- Implementar el patrón de integridad referencial manual entre dos motores.

---

## Requisitos previos

Tu PostgreSQL y tu login JWT de PSP (Tema 2 — Programación Segura) ya funcionando — vas a usar `Principal` para identificar al autor de cada reseña.

---

## Paso 1 — MongoDB en tu proyecto

Añade el servicio a `.devcontainer/docker-compose.yml` — el mismo fichero que ya tienes desde la Actividad 1.1, junto a `app` y `postgres`:

```yaml
  mongodb:
    image: mongo:8
    ports:
      - "27017:27017"
    volumes:
      - mongo_data:/data/db
```

Añade el nuevo servicio dentro de `services:`, junto a los que ya tienes, y `mongo_data:` como una entrada más dentro de `volumes:`.

En tu `pom.xml`:

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-mongodb</artifactId>
</dependency>
```

Y en `application-dev.yml`:

```yaml
spring:
  mongodb:
    uri: mongodb://mongodb:27017/gamevault_db
```

Otra vez el mismo motivo de la Actividad 1.1 con PostgreSQL: `mongodb` es el nombre del servicio, no `localhost` — tu aplicación sigue corriendo dentro del contenedor `app`, y `mongodb` es ahora un tercer contenedor hermano en la misma red.

Reconstruye tu Dev Container desde tu editor para que vuelva a procesar `docker-compose.yml` y levante también el nuevo servicio `mongodb`.

---

## Paso 2 — La entidad `Review`, su repositorio y sus DTOs

Las reseñas son una funcionalidad nueva y autocontenida, así que le toca su propio paquete: `reviews`, creado hoy. La entidad, en `src/main/java/com/tunombre/gamevault/reviews/Review.java`:

```java
package com.tunombre.gamevault.reviews;

import lombok.Data;
import lombok.NoArgsConstructor;
import org.springframework.data.annotation.Id;
import org.springframework.data.mongodb.core.mapping.Document;

@Document(collection = "review")
@Data
@NoArgsConstructor
public class Review {

    @Id
    private String id;

    private Long videojuegoId;
    private String autor;
    private Integer puntuacion;
    private String comentario;
}
```

El repositorio, en el mismo paquete, `ReviewRepository.java`:

```java
package com.tunombre.gamevault.reviews;

import org.springframework.data.mongodb.repository.MongoRepository;
import java.util.List;

public interface ReviewRepository extends MongoRepository<Review, String> {
    List<Review> findByVideojuegoId(Long videojuegoId);
}
```

`@Document(collection = "review")` es el equivalente Mongo de `@Entity`; el `@Id` es `String` porque Spring Data puede mapear de esta forma el `_id`/`ObjectId` generado por MongoDB. `findByVideojuegoId` se genera automáticamente por Spring Data a partir del nombre del método, sin que escribas ninguna query.

Los DTOs van en un subpaquete `dto`, igual que ya tienes en `catalogo` — `ReviewCreateDTO.java`, `ReviewRequestDTO.java`, `ReviewResponseDTO.java`, los tres en `src/main/java/com/tunombre/gamevault/reviews/dto/` (cada uno en su propio fichero, aunque aquí se muestren juntos por brevedad):

```java
package com.tunombre.gamevault.reviews.dto;

import jakarta.validation.constraints.NotNull;
import jakarta.validation.constraints.Min;
import jakarta.validation.constraints.Max;
import jakarta.validation.constraints.NotBlank;

public record ReviewCreateDTO(Long videojuegoId, String autor, Integer puntuacion, String comentario) {}
public record ReviewRequestDTO(
        @NotNull @Min(1) @Max(10) Integer puntuacion,
        @NotBlank String comentario
) {}
public record ReviewResponseDTO(String id, Long videojuegoId, String autor, Integer puntuacion, String comentario) {}
```

---

## Paso 3 — Integridad referencial manual, guiada al completo

Antes de devolver las reseñas de un videojuego, comprueba primero que ese videojuego existe de verdad en PostgreSQL — si no, no tiene sentido ni mirar en Mongo. `ReviewService`, en `src/main/java/com/tunombre/gamevault/reviews/ReviewService.java`, inyecta directamente `VideojuegoRepository`, del paquete `catalogo`:

```java
package com.tunombre.gamevault.reviews;

import com.tunombre.gamevault.catalogo.VideojuegoRepository;
import com.tunombre.gamevault.reviews.dto.ReviewResponseDTO;
import lombok.RequiredArgsConstructor;
import org.springframework.http.HttpStatus;
import org.springframework.stereotype.Service;
import org.springframework.web.server.ResponseStatusException;
import java.util.List;

@Service
@RequiredArgsConstructor
public class ReviewService {
    private final ReviewRepository reviewRepository;
    private final VideojuegoRepository videojuegoRepository;

    public List<ReviewResponseDTO> findByVideojuegoId(Long videojuegoId) {
        if (!videojuegoRepository.existsById(videojuegoId)) {
            throw new ResponseStatusException(HttpStatus.NOT_FOUND, "Videojuego no encontrado en el catálogo");
        }
        return reviewRepository.findByVideojuegoId(videojuegoId).stream().map(this::mapToDTO).toList();
    }

    private ReviewResponseDTO mapToDTO(Review r) {
        return new ReviewResponseDTO(r.getId(), r.getVideojuegoId(), r.getAutor(), r.getPuntuacion(), r.getComentario());
    }
}
```

**Pregunta de comprensión**: apoyándote en el diagrama de secuencia de la teoría, ¿por qué esta comprobación (`existsById`) no la puede hacer MongoDB por sí solo, como sí haría PostgreSQL con una clave foránea real?

---

## Paso 4 — El `POST` de reseñas, guiado

Añade el método `create` a `ReviewService`, reutilizando la misma comprobación que ya conoces del Paso 3:

```java
// En ReviewService
public ReviewResponseDTO create(ReviewCreateDTO dto) {
    if (!videojuegoRepository.existsById(dto.videojuegoId())) {
        throw new ResponseStatusException(HttpStatus.NOT_FOUND, "No puedes reseñar un juego que no existe en el catálogo");
    }
    Review review = new Review();
    review.setVideojuegoId(dto.videojuegoId());
    review.setAutor(dto.autor());
    review.setPuntuacion(dto.puntuacion());
    review.setComentario(dto.comentario());
    return mapToDTO(reviewRepository.save(review));
}
```

Y el controller, también en `reviews` — `src/main/java/com/tunombre/gamevault/reviews/VideojuegoReviewController.java`:

```java
package com.tunombre.gamevault.reviews;

import io.swagger.v3.oas.annotations.Operation;
import io.swagger.v3.oas.annotations.responses.ApiResponse;
import io.swagger.v3.oas.annotations.responses.ApiResponses;

@RestController
@RequestMapping("/api/v1/videojuegos/{videojuegoId}/reviews")
@RequiredArgsConstructor
public class VideojuegoReviewController {
    private final ReviewService reviewService;

    @Operation(summary = "Listar las reseñas de un videojuego")
    @ApiResponses({
            @ApiResponse(responseCode = "200", description = "Listado obtenido correctamente"),
            @ApiResponse(responseCode = "404", description = "Videojuego no encontrado en el catálogo")
    })
    @GetMapping
    public ResponseEntity<List<ReviewResponseDTO>> getByVideojuegoId(@PathVariable Long videojuegoId) {
        return ResponseEntity.ok(reviewService.findByVideojuegoId(videojuegoId));
    }

    @Operation(summary = "Crear una reseña para un videojuego")
    @ApiResponses({
            @ApiResponse(responseCode = "201", description = "Reseña creada correctamente"),
            @ApiResponse(responseCode = "400", description = "Puntuación o comentario inválidos"),
            @ApiResponse(responseCode = "401", description = "No autenticado"),
            @ApiResponse(responseCode = "404", description = "Videojuego no encontrado en el catálogo")
    })
    @PostMapping
    public ResponseEntity<ReviewResponseDTO> create(
            @PathVariable Long videojuegoId,
            @Valid @RequestBody ReviewRequestDTO dto,
            Principal principal
    ) {
        ReviewCreateDTO createDTO = new ReviewCreateDTO(videojuegoId, principal.getName(), dto.puntuacion(), dto.comentario());
        return ResponseEntity.status(HttpStatus.CREATED).body(reviewService.create(createDTO));
    }
}
```

Fíjate en que el autor **no** viaja en el cuerpo de la petición — se toma de `principal.getName()`, el usuario autenticado por el JWT que ya has construido en PSP. Repasa el diagrama de secuencia de la teoría (petición → filtro JWT → `SecurityContextHolder` → tu controller) si no tienes claro de dónde sale exactamente ese `Principal`.

!!! warning "Esta ruta todavía no existe en tu `SecurityConfig` — añádela ahora"
    Las reseñas no existían en la Actividad 2.5 de PSP, así que no aparecen en tu política de rutas — y con `denyAll()` cerrando todo lo que no está listado explícitamente, `/api/v1/videojuegos/*/reviews` está bloqueada hasta que la añadas tú:
    ```java
    .requestMatchers(HttpMethod.GET, "/api/v1/videojuegos/*/reviews/**").permitAll()
    .requestMatchers(HttpMethod.POST, "/api/v1/videojuegos/*/reviews").authenticated()
    ```
    `GET` queda pública, igual que el resto de lecturas del catálogo. `POST` exige estar autenticado, pero sin ningún rol concreto: quien puede escribir una reseña es cualquier usuario logueado, no solo `ADMIN` — la comprobación que de verdad importa aquí es *quién eres* (para guardarte como autor), no *qué rol tienes*.

Ahora sí, prueba con tu token. Aquí tienes los comandos con `curl`, pero puedes hacer exactamente lo mismo desde Swagger UI si lo prefieres:

```bash
curl -X POST http://localhost:8080/api/v1/videojuegos/1/reviews \
  -H "Authorization: Bearer $USER_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"puntuacion": 9, "comentario": "Excelente banda sonora"}'
```

**Captura**: la respuesta del `POST`, con el `autor` ya relleno aunque no lo hayas mandado tú en el cuerpo.

Consulta ahora el mismo videojuego con `GET`, para comprobar que la reseña que acabas de crear aparece en el listado:

```bash
curl http://localhost:8080/api/v1/videojuegos/1/reviews
```

**Captura**: la respuesta del `GET`, con la reseña recién creada dentro de la lista.

---

## Paso 5 — El endpoint de resumen

Otro DTO más en `reviews/dto/`, `ReviewResumenDTO.java`:

```java
package com.tunombre.gamevault.reviews.dto;

public record ReviewResumenDTO(Long videojuegoId, long totalReviews, double puntuacionMedia) {}
```

Sin más código dado, completa en `ReviewService` un método `getResumenByVideojuegoId(Long videojuegoId)` que: compruebe primero que el videojuego existe (mismo patrón que ya has usado dos veces), obtenga sus reseñas con `findByVideojuegoId`, y calcule con streams (`mapToInt(...).average()`) el total y la puntuación media. Expón el resultado en `GET /api/v1/videojuegos/{videojuegoId}/reviews/resumen`, documentado con `@Operation`/`@ApiResponses` igual que los otros dos endpoints del controller.

**Captura**: la respuesta de ese endpoint, con el total y la puntuación media ya calculados.

---

## Experimento de cierre — reseñas huérfanas

Crea un videojuego, añádele un par de reseñas, y bórralo desde tu API normal:

```bash
curl -X DELETE http://localhost:8080/api/v1/videojuegos/{id} \
  -H "Authorization: Bearer $ADMIN_TOKEN"
```

(recuerda: `DELETE /api/v1/videojuegos/{id}` exige rol `ADMIN` desde la Actividad 2.5 de PSP)

Consulta MongoDB directamente, desde la misma terminal integrada (gracias al `docker-outside-of-docker` de la Actividad 1.1). Busca el nombre de tu contenedor de Mongo con `docker ps` y sustitúyelo abajo:

```bash
docker exec -it <tu-contenedor-mongo> mongosh gamevault_db --eval "db.review.find({videojuegoId: <id>})"
```

**Comprueba**: las reseñas siguen ahí, apuntando a un `videojuegoId` que ya no existe en PostgreSQL — son **reseñas huérfanas**.

**Captura**: la salida de `mongosh` mostrando esas reseñas huérfanas.

Describe el problema con tus palabras: ¿qué implicaciones tiene tener datos en Mongo que referencian algo que ya no existe en Postgres? Este es exactamente el problema que vas a abordar en la próxima actividad.

---

## ✅ Cierre

Tu GameVault ya habla con dos motores de base de datos distintos, con el patrón de integridad referencial manual resuelto. En la próxima actividad trabajas con colecciones explícitas y conectas el borrado en cascada de reseñas.
