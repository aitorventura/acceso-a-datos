# 🧪 Actividad 4.1: Reseñas de videojuegos en MongoDB

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
services:
  # ... tus servicios app y postgres ya existentes ...
  mongodb:
    image: mongo:8
    ports:
      - "27017:27017"
    volumes:
      - mongo_data:/data/db

volumes:
  postgres_data:
  mongo_data:
```

En tu `pom.xml`:

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-mongodb</artifactId>
</dependency>
```

Y en `application-dev.yaml`:

```yaml
spring:
  mongodb:
    uri: mongodb://mongodb:27017/gamevault_db
```

Otra vez el mismo motivo de la Actividad 1.1 con PostgreSQL: `mongodb` es el nombre del servicio, no `localhost` — tu aplicación sigue corriendo dentro del contenedor `app`, y `mongodb` es ahora un tercer contenedor hermano en la misma red.

Levanta el servicio nuevo desde la terminal integrada — comprueba primero el nombre real de tu proyecto con `docker compose ls` (no siempre es `gamevault_devcontainer`, depende de tu editor) y sustitúyelo en `docker compose -f .devcontainer/docker-compose.yml -p <proyecto> up -d mongodb` — y reinicia tu aplicación. Si prefieres, también puedes hacer **"Dev Containers: Rebuild Container"** desde la paleta de comandos, que relee el `docker-compose.yml` completo y levanta el servicio nuevo por ti.

---

## Paso 2 — La entidad `Review` y su repositorio

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

```java
package com.tunombre.gamevault.reviews;

import org.springframework.data.mongodb.repository.MongoRepository;
import java.util.List;

public interface ReviewRepository extends MongoRepository<Review, String> {
    List<Review> findByVideojuegoId(Long videojuegoId);
}
```

`@Document(collection = "review")` es el equivalente Mongo de `@Entity`; el `@Id` es `String` porque en Mongo los identificadores son alfanuméricos. `findByVideojuegoId` se genera automáticamente por Spring Data a partir del nombre del método, sin que escribas ninguna query.

---

## Paso 3 — Integridad referencial manual, guiada al completo

Antes de usarlo desde `reviews`, crea el pequeño componente que va a permitir esa comprobación sin que `reviews` conozca nada del paquete `catalogo` por dentro — una interfaz en `catalogo.api` y su implementación oculta en `catalogo`:

```java
package com.tunombre.gamevault.catalogo.api;

public interface CatalogoConsultaService {
    boolean existeVideojuego(Long videojuegoId);
}
```

```java
package com.tunombre.gamevault.catalogo;

import com.tunombre.gamevault.catalogo.api.CatalogoConsultaService;
import lombok.RequiredArgsConstructor;
import org.springframework.stereotype.Service;

@Service
@RequiredArgsConstructor
class CatalogoConsultaServiceImpl implements CatalogoConsultaService {

    private final VideojuegoRepository videojuegoRepository;

    @Override
    public boolean existeVideojuego(Long videojuegoId) {
        return videojuegoRepository.existsById(videojuegoId);
    }
}
```

Fíjate en dos detalles deliberados: la interfaz vive en `catalogo.api` (un paquete pensado para lo que otros módulos SÍ pueden ver), mientras que `CatalogoConsultaServiceImpl` no lleva el modificador `public` — es *package-private*, invisible fuera de `catalogo`. El módulo `reviews` va a depender solo de la interfaz, sin saber nada de cómo está implementada por dentro. Vas a profundizar en por qué esto importa en el Tema 5 — de momento, réplicalo tal cual.

Ahora sí, el service de `reviews` que lo usa:

```java
package com.tunombre.gamevault.reviews;

import com.tunombre.gamevault.catalogo.api.CatalogoConsultaService;
import lombok.RequiredArgsConstructor;
import org.springframework.http.HttpStatus;
import org.springframework.stereotype.Service;
import org.springframework.web.server.ResponseStatusException;
import java.util.List;

@Service
@RequiredArgsConstructor
public class ReviewService {
    private final ReviewRepository reviewRepository;
    private final CatalogoConsultaService catalogoConsultaService;

    public List<ReviewResponseDTO> findByVideojuegoId(Long videojuegoId) {
        if (!catalogoConsultaService.existeVideojuego(videojuegoId)) {
            throw new ResponseStatusException(HttpStatus.NOT_FOUND, "Videojuego no encontrado en el catálogo");
        }
        return reviewRepository.findByVideojuegoId(videojuegoId).stream().map(this::mapToDTO).toList();
    }

    private ReviewResponseDTO mapToDTO(Review r) {
        return new ReviewResponseDTO(r.getId(), r.getVideojuegoId(), r.getAutor(), r.getPuntuacion(), r.getComentario());
    }
}
```

**Pregunta de comprensión**: ¿por qué esta comprobación (`existeVideojuego`) no la puede hacer MongoDB por sí solo, como sí haría PostgreSQL con una clave foránea real?

---

## Paso 4 — El `POST` de reseñas, guiado

```java
public record ReviewCreateDTO(Long videojuegoId, String autor, Integer puntuacion, String comentario) {}
public record ReviewRequestDTO(
        @jakarta.validation.constraints.Min(1) @jakarta.validation.constraints.Max(10) Integer puntuacion,
        @jakarta.validation.constraints.NotBlank String comentario
) {}
public record ReviewResponseDTO(String id, Long videojuegoId, String autor, Integer puntuacion, String comentario) {}
```

```java
// En ReviewService
public ReviewResponseDTO create(ReviewCreateDTO dto) {
    if (!catalogoConsultaService.existeVideojuego(dto.videojuegoId())) {
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

```java
@RestController
@RequestMapping("/api/v1/videojuegos/{videojuegoId}/reviews")
@RequiredArgsConstructor
public class VideojuegoReviewController {
    private final ReviewService reviewService;

    @GetMapping
    public ResponseEntity<List<ReviewResponseDTO>> getByVideojuegoId(@PathVariable Long videojuegoId) {
        return ResponseEntity.ok(reviewService.findByVideojuegoId(videojuegoId));
    }

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

Fíjate en que el autor **no** viaja en el cuerpo de la petición — se toma de `principal.getName()`, el usuario autenticado por el JWT que ya construiste en PSP. Prueba con tu token:

```bash
curl -X POST http://localhost:8080/api/v1/videojuegos/1/reviews \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"puntuacion": 9, "comentario": "Excelente banda sonora"}'
```

---

## Mini-reto — el endpoint de resumen

```java
public record ReviewResumenDTO(Long videojuegoId, long totalReviews, double puntuacionMedia) {}
```

Sin más código dado, completa en `ReviewService` un método `getResumenByVideojuegoId(Long videojuegoId)` que: compruebe primero que el videojuego existe (mismo patrón que ya has usado dos veces), obtenga sus reseñas con `findByVideojuegoId`, y calcule con streams (`mapToInt(...).average()`) el total y la puntuación media. Expón el resultado en `GET /api/v1/videojuegos/{videojuegoId}/reviews/resumen`.

---

## Experimento de cierre — reseñas huérfanas

Crea un videojuego, añádele un par de reseñas, y bórralo desde tu API normal:

```bash
curl -X DELETE http://localhost:8080/api/v1/videojuegos/{id}
```

Consulta MongoDB directamente, desde la misma terminal integrada (gracias al `docker-outside-of-docker` de la Actividad 1.1). Comprueba primero el nombre real de tu proyecto con `docker compose ls` (no siempre es `gamevault_devcontainer`) y sustitúyelo por `<proyecto>`:

```bash
docker compose -f .devcontainer/docker-compose.yml -p <proyecto> ps
docker exec -it <tu-contenedor-mongo> mongosh gamevault_db --eval "db.review.find({videojuegoId: <id>})"
```

**Comprueba**: las reseñas siguen ahí, apuntando a un `videojuegoId` que ya no existe en PostgreSQL — son **reseñas huérfanas**. Describe el problema con tus palabras: ¿qué implicaciones tiene tener datos en Mongo que referencian algo que ya no existe en Postgres? Este es exactamente el problema que vas a abordar en la próxima actividad.

---

## ✅ Cierre

Tu GameVault ya habla con dos motores de base de datos distintos, con el patrón de integridad referencial manual resuelto. En la próxima actividad trabajas con colecciones explícitas y conectas el borrado en cascada de reseñas.
