<a id="modificacion-documentos"></a>

# 🧩 3. Modificación de documentos

Te falta la pieza final del módulo `reviews`: modificar un documento concreto, con una particularidad que no habías tenido que resolver hasta ahora — comprobar quién tiene permiso para hacerlo.

---

## ✏️ Añadir, modificar y eliminar documentos, en `mongosh`

```javascript
// Añadir
db.review.insertOne({ videojuegoId: 1, autor: "ana", puntuacion: 8 })

// Modificar solo un campo concreto
db.review.updateOne({ _id: ObjectId("...") }, { $set: { puntuacion: 9 } })

// Reemplazar el documento entero
db.review.replaceOne({ _id: ObjectId("...") }, { videojuegoId: 1, autor: "ana", puntuacion: 9, comentario: "..." })

// Eliminar
db.review.deleteOne({ _id: ObjectId("...") })
```

Hay una diferencia conceptual importante entre `updateOne` con `$set` (modifica solo los campos indicados, deja el resto intacto) y `replaceOne` (sustituye el documento entero). Conviene que sepas que `$set` existe, aunque el proyecto no lo use directamente: `save()` de Spring Data MongoDB, que ya conoces, hace lo segundo — reemplaza el documento completo, no un *merge* parcial.

---

## 🎮 Aterrizaje en GameVault: el `PUT` que falta, con control de autoría

En el estado actual del proyecto, `VideojuegoReviewController` solo expone `GET` (listar), `POST` (crear) y `GET .../resumen` — no existe un `PUT` para modificar una reseña ya publicada. Es la pieza que vas a añadir, con una particularidad importante: **control de autoría** — solo el autor original de una reseña debería poder modificarla, no cualquier usuario autenticado.

### De dónde sale la identidad: `Principal`

```java
@PostMapping
public ResponseEntity<ReviewResponseDTO> create(
        @PathVariable Long videojuegoId,
        @Valid @RequestBody ReviewRequestDTO dto,
        Principal principal
) {
    ReviewCreateDTO createDTO = new ReviewCreateDTO(videojuegoId, principal.getName(), dto.puntuacion(), dto.comentario());
    return ResponseEntity.status(HttpStatus.CREATED).body(reviewService.create(createDTO));
}
```

El `POST` que ya conoces usa `Principal principal` para tomar `principal.getName()` como autor — es la identidad del usuario autenticado, inyectada automáticamente por Spring Security a partir del JWT que construiste en Programación de Servicios y Procesos (usuarios, roles, JWT). No hace falta que repases aquí cómo funciona ese JWT por dentro — solo que sepas que, si tu petición llega autenticada, `Principal` te da el nombre de quien la hizo.

### El patrón de control de autoría

```java
public ReviewResponseDTO update(String reviewId, ReviewRequestDTO dto, String usuarioActual) {
    Review review = reviewRepository.findById(reviewId)
            .orElseThrow(() -> new ResponseStatusException(HttpStatus.NOT_FOUND, "Reseña no encontrada"));

    if (!review.getAutor().equals(usuarioActual)) {
        throw new ResponseStatusException(HttpStatus.FORBIDDEN, "Solo el autor puede modificar esta reseña");
    }

    review.setPuntuacion(dto.puntuacion());
    review.setComentario(dto.comentario());

    return mapToDTO(reviewRepository.save(review));
}
```

El patrón, en tres pasos: cargar el documento por id, comprobar que el campo `autor` guardado coincide con `usuarioActual` (el `principal.getName()` de la petición), y solo entonces aplicar los cambios y guardar. Si no coincide, `403 Forbidden` — el usuario está autenticado (sabemos quién es), pero no tiene permiso para esta acción concreta sobre este recurso. Si la reseña no existe, `404 Not Found`, como ya conoces.

### `save()` sirve para crear y para actualizar — otra vez

`ReviewRepository` (que extiende `MongoRepository<Review, String>`) ya tiene `save()` heredado, y sirve tanto para crear un documento nuevo como para actualizar uno existente — basta con que el objeto que le pases tenga el mismo `id` que ya existe en la colección. No es un concepto nuevo: es exactamente lo mismo que ya viste con `JpaRepository` en el Tema 2 — solo cambia el motor de base de datos por debajo, el comportamiento de `save()` es idéntico en la forma de razonar sobre él.

---

## 🧭 Recapitulación del tema

Con esto se completa el recorrido: conexión y consultas sobre MongoDB (apartado 1) → comparación relacional/documental y gestión de colecciones (apartado 2) → modificación de documentos con control de acceso (este apartado). El Tema 5 retoma `CatalogoConsultaService`/`CatalogoConsultaServiceImpl` — que ya usaste aquí mismo para la integridad referencial manual — como el ejemplo central de qué es un **componente** de acceso a datos.

---

## ✅ Ideas clave

??? tip "Abrir resumen"

    - `updateOne` con `$set` modifica campos concretos; `replaceOne` (y `save()` de Spring Data) sustituye el documento entero.
    - `Principal` te da la identidad del usuario autenticado, inyectada por Spring Security a partir del JWT construido en PSP.
    - El patrón de control de autoría: cargar → comprobar que el autor coincide con el usuario actual → solo entonces modificar y guardar. `403` si no coincide, `404` si no existe.
    - `save()` de `MongoRepository` sirve tanto para crear como para actualizar, exactamente igual que `JpaRepository` — mismo concepto, distinto motor.
