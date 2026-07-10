# 🧪 Actividad 1.2: CRUD completo y DTOs sobre el catálogo

!!! info "Práctica guiada"
    Vas a construir, sobre las entidades que creaste en la Actividad 1.1, el CRUD completo de `Videojuego`: DTOs, controller y service con transacciones. Al terminar, tu GameVault sabrá crear, leer, modificar y borrar videojuegos de verdad.

## Qué vas a practicar

- Definir DTOs de entrada y salida como `record`, distintos de la entidad JPA.
- Construir un controller REST completo (GET, POST, PUT, DELETE).
- Aplicar `@Transactional` correctamente en operaciones de escritura y de lectura.
- Gestionar el caso "no encontrado" con `ResponseStatusException`.

!!! warning "Qué NO incluye esta actividad todavía"
    No incluyas paginación con `Pageable`, el filtro `VideojuegoFiltroDTO` ni `Specifications` — eso llega en el Tema 2. Tampoco incluyas ninguna llamada a un `EventPublisher`: la versión completa de GameVault publica eventos de mensajería tras cada operación, pero eso es contenido de Programación de Servicios y Procesos más adelante en el curso. Omite esas líneas por completo en tu propio código por ahora.

---

## Requisitos previos

Tu proyecto de la Actividad 1.1, con las entidades `Estudio` y `Videojuego` ya creadas y verificadas contra la base de datos.

---

## Paso 1 — Los DTOs

Crea el paquete `catalogo.dto` con dos records:

```java
package com.tunombre.gamevault.catalogo.dto;

import jakarta.validation.constraints.*;
import java.math.BigDecimal;
import java.time.LocalDate;

public record VideojuegoCreateDTO(
        @NotBlank(message = "El título no puede estar vacío")
        @Size(max = 150)
        String titulo,

        @NotNull
        @PositiveOrZero(message = "El precio no puede ser negativo")
        BigDecimal precio,

        @NotNull
        @PastOrPresent(message = "La fecha de lanzamiento no puede ser futura")
        LocalDate fechaLanzamiento,

        @NotNull
        Long estudioId
) {}
```

```java
package com.tunombre.gamevault.catalogo.dto;

import java.math.BigDecimal;
import java.time.LocalDate;

public record VideojuegoResponseDTO(
        Long id,
        String titulo,
        BigDecimal precio,
        LocalDate fechaLanzamiento,
        String nombreEstudio
) {}
```

Fíjate en la diferencia entre los dos: `VideojuegoCreateDTO` pide `estudioId` (un simple `Long`, lo mínimo necesario para relacionar) y lleva las anotaciones de validación (`@NotBlank`, `@PositiveOrZero`...) que se comprobarán cuando actives `@Valid` en el controller; `VideojuegoResponseDTO` no lleva validación (nadie valida lo que tu propia aplicación devuelve) y en vez de `estudioId` lleva `nombreEstudio`, un dato ya "aplanado" y listo para mostrar.

**Pregunta**: ¿por qué `VideojuegoResponseDTO` no tiene sentido que lleve anotaciones `@NotBlank`/`@NotNull` como el DTO de creación?

---

## Paso 2 — GET y POST, guiados al completo

En `VideojuegoService`, empieza por el mapeo y las operaciones de lectura:

```java
@Service
@RequiredArgsConstructor
public class VideojuegoService {
    private final VideojuegoRepository videojuegoRepository;
    private final EstudioRepository estudioRepository;

    @Transactional(readOnly = true)
    public List<VideojuegoResponseDTO> findAll() {
        return videojuegoRepository.findAll()
                .stream()
                .map(this::mapToDTO)
                .toList();
    }

    @Transactional(readOnly = true)
    public VideojuegoResponseDTO findById(Long id) {
        return videojuegoRepository.findById(id)
                .map(this::mapToDTO)
                .orElseThrow(() -> new ResponseStatusException(HttpStatus.NOT_FOUND, "Videojuego no encontrado"));
    }

    @Transactional
    public VideojuegoResponseDTO create(VideojuegoCreateDTO dto) {
        Estudio estudio = estudioRepository.findById(dto.estudioId())
                .orElseThrow(() -> new ResponseStatusException(HttpStatus.NOT_FOUND, "Estudio no encontrado"));

        Videojuego v = new Videojuego();
        v.setTitulo(dto.titulo());
        v.setPrecio(dto.precio());
        v.setFechaLanzamiento(dto.fechaLanzamiento());
        v.setEstudio(estudio);

        return mapToDTO(videojuegoRepository.save(v));
    }

    private VideojuegoResponseDTO mapToDTO(Videojuego v) {
        return new VideojuegoResponseDTO(
                v.getId(), v.getTitulo(), v.getPrecio(), v.getFechaLanzamiento(), v.getEstudio().getNombre()
        );
    }
}
```

`findById` y `create` siguen el mismo patrón que ya viste en la teoría: `readOnly = true` en la lectura, transacción normal en la escritura, `orElseThrow` con `ResponseStatusException(HttpStatus.NOT_FOUND, ...)` cuando algo no existe — tanto para el videojuego como, en `create`, para el estudio al que debe pertenecer.

Ahora el controller:

```java
@RestController
@RequestMapping("/api/v1/videojuegos")
@RequiredArgsConstructor
public class VideojuegoController {
    private final VideojuegoService videojuegoService;

    @GetMapping
    public ResponseEntity<List<VideojuegoResponseDTO>> getAll() {
        return ResponseEntity.ok(videojuegoService.findAll());
    }

    @GetMapping("/{id}")
    public ResponseEntity<VideojuegoResponseDTO> getById(@PathVariable Long id) {
        return ResponseEntity.ok(videojuegoService.findById(id));
    }

    @PostMapping
    public ResponseEntity<VideojuegoResponseDTO> create(@Valid @RequestBody VideojuegoCreateDTO dto) {
        return ResponseEntity.status(HttpStatus.CREATED).body(videojuegoService.create(dto));
    }
}
```

Arranca tu aplicación y prueba, con `curl` o Postman:

```bash
curl -X POST http://localhost:8080/api/v1/videojuegos \
  -H "Content-Type: application/json" \
  -d '{"titulo":"Hades","precio":24.99,"fechaLanzamiento":"2020-09-17","estudioId":1}'
```

**Predicción**: si `estudioId` apunta a un estudio que no existe en tu base de datos, ¿qué código de estado esperas recibir? Compruébalo después de escribir tu respuesta.

---

## Paso 3 — El PUT, guiado al completo

El `PUT` es la operación con más matices: hay que cargar el recurso existente, comprobar que existe, modificar sus campos y guardar — no crear uno nuevo.

```java
// En VideojuegoService
@Transactional
public VideojuegoResponseDTO update(Long id, VideojuegoCreateDTO dto) {
    Videojuego v = videojuegoRepository.findById(id)
            .orElseThrow(() -> new ResponseStatusException(HttpStatus.NOT_FOUND, "Videojuego no encontrado"));

    Estudio estudio = estudioRepository.findById(dto.estudioId())
            .orElseThrow(() -> new ResponseStatusException(HttpStatus.NOT_FOUND, "Estudio no encontrado"));

    v.setTitulo(dto.titulo());
    v.setPrecio(dto.precio());
    v.setFechaLanzamiento(dto.fechaLanzamiento());
    v.setEstudio(estudio);

    return mapToDTO(videojuegoRepository.save(v));
}
```

```java
// En VideojuegoController
@PutMapping("/{id}")
public ResponseEntity<VideojuegoResponseDTO> update(@PathVariable Long id, @Valid @RequestBody VideojuegoCreateDTO dto) {
    return ResponseEntity.ok(videojuegoService.update(id, dto));
}
```

Fíjate en que `update` reutiliza el mismo `VideojuegoCreateDTO` que `create` — no necesitas un DTO distinto para actualizar, porque los datos que se piden son los mismos. Nota también que, aunque el objeto `v` ya está siendo gestionado por Hibernate (viene de un `findById` dentro de la misma transacción), el `save()` explícito al final no sobra: deja claro en el código, sin ambigüedad, cuál es el punto en el que se persiste el cambio.

---

## Mini-reto — el DELETE

Repite el patrón que ya has usado dos veces (cargar + comprobar existencia + actuar). Sin más pistas que estas:

- El método del service debe devolver `void`.
- El endpoint debe responder `204 No Content` si todo va bien.
- Si el `id` no existe, debe comportarse igual que `findById`/`update` ante ese mismo caso.

Complétalo tú mismo en `VideojuegoService` y en `VideojuegoController`.

---

## Paso 4 — Comprobación final de las cuatro operaciones

```bash
# Crear
curl -X POST http://localhost:8080/api/v1/videojuegos -H "Content-Type: application/json" \
  -d '{"titulo":"Hades","precio":24.99,"fechaLanzamiento":"2020-09-17","estudioId":1}'

# Leer
curl http://localhost:8080/api/v1/videojuegos

# Actualizar (sustituye 1 por el id real que te haya devuelto el create)
curl -X PUT http://localhost:8080/api/v1/videojuegos/1 -H "Content-Type: application/json" \
  -d '{"titulo":"Hades","precio":19.99,"fechaLanzamiento":"2020-09-17","estudioId":1}'

# Borrar
curl -X DELETE http://localhost:8080/api/v1/videojuegos/1
```

**Comprueba**, para cada una, el código de estado devuelto y que coincide con lo esperado según lo visto en la teoría.

---

## Pregunta final

`create()` tiene una sola operación de guardado (`videojuegoRepository.save(v)`). ¿Qué aporta exactamente `@Transactional` en ese caso, si solo hay una escritura? ¿Y si mañana ese mismo método tuviera que guardar el videojuego *y* actualizar un contador en otra tabla — cambiaría en algo la respuesta?

---

## ✅ Cierre

Tu GameVault ya tiene un CRUD completo y transaccional sobre `Videojuego`. Todavía lo has hecho todo con Spring Data JPA gestionando la conexión por ti — en la próxima actividad vas a ver, con JDBC puro, toda la "fontanería" que hay debajo de ese `@Transactional` que acabas de usar.
