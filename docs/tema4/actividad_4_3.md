# 🧪 Actividad 4.3: PUT de reseñas con control de autoría

!!! info "Práctica guiada"
    Hoy añades el `PUT` que falta en tu módulo `reviews`, con control de autoría: solo quien escribió una reseña puede modificarla.

## Qué vas a practicar

- Añadir un endpoint `PUT` que compruebe la identidad del solicitante.
- Aplicar el patrón cargar → comprobar autoría → modificar → guardar.
- Verificar con dos usuarios distintos que el control de acceso funciona de verdad.

---

## Requisitos previos

Tu módulo `reviews` (Actividades 4.1-4.2) y tu login JWT de PSP ya funcionando — vas a necesitar dos usuarios de prueba distintos.

---

## Paso 1 — El endpoint, guiado al completo

```java
// En ReviewService
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

```java
// En VideojuegoReviewController
@PutMapping("/{reviewId}")
public ResponseEntity<ReviewResponseDTO> update(
        @PathVariable Long videojuegoId,
        @PathVariable String reviewId,
        @Valid @RequestBody ReviewRequestDTO dto,
        Principal principal
) {
    return ResponseEntity.ok(reviewService.update(reviewId, dto, principal.getName()));
}
```

Igual que ya hace el `POST`, recibes `Principal principal` y le pasas `principal.getName()` al service — la comparación de autoría ocurre dentro de `update()`, no en el controller.

---

## Paso 2 — Probar con dos usuarios: el caso correcto

Crea una reseña con el usuario `user` (o el que uses habitualmente) y anota el `id` que te devuelve MongoDB:

```bash
curl -X POST http://localhost:8080/api/v1/videojuegos/1/reviews \
  -H "Authorization: Bearer $TOKEN_USER" -H "Content-Type: application/json" \
  -d '{"puntuacion": 7, "comentario": "Buena, pero corta"}'
```

Modifícala con el **mismo** usuario:

```bash
curl -X PUT http://localhost:8080/api/v1/videojuegos/1/reviews/{reviewId} \
  -H "Authorization: Bearer $TOKEN_USER" -H "Content-Type: application/json" \
  -d '{"puntuacion": 8, "comentario": "Buena, pero corta — la he rejugado y mejora"}'
```

**Comprueba**: `200 OK`, con los datos actualizados en la respuesta.

---

## Paso 3 — Probar con dos usuarios: el caso denegado

Consigue un token de un **segundo** usuario (crea uno nuevo si solo tienes uno de prueba) e intenta modificar la reseña del primero:

```bash
curl -i -X PUT http://localhost:8080/api/v1/videojuegos/1/reviews/{reviewId} \
  -H "Authorization: Bearer $TOKEN_OTRO_USUARIO" -H "Content-Type: application/json" \
  -d '{"puntuacion": 1, "comentario": "Intento de modificación ajena"}'
```

**Comprueba**: `403 Forbidden`. **Documenta** ambos resultados (Paso 2 y este) con el código de estado y el cuerpo de cada respuesta.

---

## Pregunta de comprensión

¿Por qué la comprobación de autoría devuelve `403 Forbidden` y no `404 Not Found`, sabiendo que la reseña sí existe? ¿Cambiaría tu respuesta si, en vez de una reseña, estuvieras protegiendo un recurso donde no quieres ni confirmar que existe a alguien sin permiso? (No hace falta que cambies el código — solo que razones la diferencia).

---

## Resumen del tema

Escribe un resumen propio (5-6 líneas) del recorrido completo de este tema: conectar y consultar MongoDB por primera vez (Actividad 4.1) → entender colecciones y el borrado en cascada vía eventos (Actividad 4.2) → modificar documentos con control de acceso real (esta actividad). ¿Qué diferencia concreta has notado entre trabajar con PostgreSQL (Temas 1-3) y con MongoDB (este tema) — en qué momento has echado en falta las garantías del modelo relacional, y en qué momento has agradecido la flexibilidad del documental?

---

## ✅ Cierre

Con esto termina todo el bloque de bases de datos del módulo. En el Tema 5, el último del curso, das un paso atrás para ver todo lo construido bajo una óptica distinta: componentes reutilizables, con `CatalogoConsultaService` (ya usado aquí mismo) como ejemplo central.
