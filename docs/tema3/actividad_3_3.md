# 🧪 Actividad 3.3: PUT y DELETE de reseñas con control de autoría

!!! warning "Descarga la plantilla"
    📄 [Plantilla 3.3 — PUT y DELETE de reseñas con control de autoría](plantillas/Actividad_3_3_AD_Plantilla.docx){target="_blank" rel="noopener"}

!!! info "Práctica guiada"
    Hoy añades el `PUT` que falta en tu módulo `reviews`, con control de autoría: solo quien escribió una reseña puede modificarla.

## Qué vas a practicar

- Añadir un endpoint `PUT` y un endpoint `DELETE` que comprueben la identidad del solicitante.
- Aplicar el patrón cargar → comprobar autoría (o rol de administrador) → actuar.
- Verificar con dos usuarios distintos, y con un administrador, que el control de acceso funciona de verdad.

---

## Requisitos previos

Tu módulo `reviews` (Actividades 3.1-3.2) y tu login JWT de PSP ya funcionando — vas a necesitar dos usuarios de prueba distintos y un token de administrador.

---

## Paso 1 — El endpoint, guiado al completo

```java
// En ReviewService
public ReviewResponseDTO update(Long videojuegoId, String reviewId, ReviewRequestDTO dto, String usuarioActual, boolean esAdmin) {
    // 1. Carga la reseña por su id — si no existe, 404 Not Found
    //    (mismo patrón que acabas de ver en la teoría de este apartado)

    // 2. Comprueba que esa reseña pertenece de verdad a videojuegoId — si no, 404 Not Found
    //    (mismo motivo que ya has visto en la teoría: el id de la reseña ya es único,
    //    pero la URL afirma que pertenece a este videojuego en concreto)

    // 3. Comprueba que usuarioActual sea el autor, o que esAdmin sea true.
    //    Si ninguna de las dos se cumple, 403 Forbidden

    review.setPuntuacion(dto.puntuacion());
    review.setComentario(dto.comentario());

    return mapToDTO(reviewRepository.save(review));
}
```

Sin más código dado que el que ya ves aquí: escribe tú los pasos que faltan, siguiendo el patrón de la teoría de este apartado — cambia solo `NotaLectura`/`notaLecturaRepository` por `Review`/`reviewRepository`, y añade las condiciones de `videojuegoId` y `esAdmin`.

```java
// En VideojuegoReviewController
@Operation(summary = "Modificar una reseña (solo el autor o un administrador)")
@ApiResponses({
        @ApiResponse(responseCode = "200", description = "Reseña actualizada correctamente"),
        @ApiResponse(responseCode = "400", description = "Puntuación o comentario inválidos"),
        @ApiResponse(responseCode = "401", description = "No autenticado"),
        @ApiResponse(responseCode = "403", description = "No eres el autor ni un administrador"),
        @ApiResponse(responseCode = "404", description = "Reseña no encontrada")
})
@PutMapping("/{reviewId}")
public ResponseEntity<ReviewResponseDTO> update(
        @PathVariable Long videojuegoId,
        @PathVariable String reviewId,
        @Valid @RequestBody ReviewRequestDTO dto,
        Authentication authentication
) {
    boolean esAdmin = authentication.getAuthorities().stream()
            .anyMatch(a -> a.getAuthority().equals("ROLE_ADMIN"));
    return ResponseEntity.ok(reviewService.update(videojuegoId, reviewId, dto, authentication.getName(), esAdmin));
}
```

`Authentication` extiende `Principal` — `authentication.getName()` te da lo mismo que ya conocías de `principal.getName()`, pero además expone `getAuthorities()`, con los roles del usuario (el mismo mecanismo que ya usa `JwtService`/`AuthController` para meter los roles en el JWT). La comparación de autoría — o de rol — ocurre dentro de `update()`, no en el controller: el controller solo se encarga de averiguar quién pregunta y con qué rol, no de decidir si puede o no.

!!! tip "Por qué el `ADMIN` también puede, y no solo el autor"
    Un moderador tiene que poder corregir o retirar contenido problemático sin depender de que el autor original coopere — es el mismo motivo por el que `ROLE_ADMIN` ya puede crear, editar y borrar videojuegos y estudios directamente, saltándose cualquier otra condición. Aquí el rol no sustituye a la comprobación de autoría: se **añade** como una segunda vía de permiso, `autor == usuarioActual` **o** `esAdmin`.

!!! tip "Por qué comprobar también que la reseña pertenece a `videojuegoId`"
    El `id` de la reseña ya es único por sí solo — no necesitarías `videojuegoId` para encontrarla. Pero la URL afirma que esa reseña pertenece a ese videojuego en concreto, así que conviene comprobarlo, igual que ya has visto en la teoría de este apartado.

!!! warning "Añade también la regla del `PUT` a tu `SecurityConfig`"
    Igual que has tenido que añadir la regla del `POST` en la Actividad 3.1, esta ruta nueva tampoco existe todavía en tu política:
    ```java
    .requestMatchers(HttpMethod.PUT, "/api/v1/videojuegos/*/reviews/*").authenticated()
    ```
    Otra vez basta con estar autenticado, sin rol concreto — quién puede modificar **esta** reseña en particular ya lo decide `update()` comparando el autor, no `SecurityConfig`.

---

## Paso 2 — Probar con dos usuarios: el caso correcto

Crea una reseña con el usuario `user` (o el que uses habitualmente) y anota el `id` que te devuelve MongoDB. Aquí tienes los comandos con `curl`, pero puedes hacer exactamente lo mismo desde Swagger UI si lo prefieres:

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

**Captura**: la respuesta del `PUT` con `200 OK` y los datos ya actualizados.

---

## Paso 3 — Probar con dos usuarios: el caso denegado

Consigue un token de un **segundo** usuario (crea uno nuevo si solo tienes uno de prueba) e intenta modificar la reseña del primero:

```bash
curl -i -X PUT http://localhost:8080/api/v1/videojuegos/1/reviews/{reviewId} \
  -H "Authorization: Bearer $TOKEN_OTRO_USUARIO" -H "Content-Type: application/json" \
  -d '{"puntuacion": 1, "comentario": "Intento de modificación ajena"}'
```

**Comprueba**: `403 Forbidden`.

**Captura**: la respuesta del `PUT` con `403 Forbidden`.

**Documenta** ambos resultados (Paso 2 y este) con el código de estado y el cuerpo de cada respuesta.

---

## Paso 4 — Probar con un administrador: el rol también permite

Con `$TOKEN_ADMIN`, modifica la misma reseña del Paso 3 — sin ser el autor:

```bash
curl -X PUT http://localhost:8080/api/v1/videojuegos/1/reviews/{reviewId} \
  -H "Authorization: Bearer $TOKEN_ADMIN" -H "Content-Type: application/json" \
  -d '{"puntuacion": 5, "comentario": "Moderado por un administrador"}'
```

**Comprueba**: `200 OK` — a diferencia del Paso 3, aunque tampoco sea el autor.

**Captura**: la respuesta del `PUT` con `200 OK`, hecha con `$TOKEN_ADMIN`.

---

## Paso 5 — El `DELETE` que falta, con el mismo patrón

Hasta ahora un autor puede crear y modificar su reseña, pero nunca borrarla — cierra ese hueco reutilizando exactamente el mismo patrón de permiso que acabas de aplicar en el `PUT`, esta vez sobre `delete()`. Sin código dado esta vez — es la misma estructura que ya has escrito en el Paso 1, cambiando solo qué haces al final:

```java
// En ReviewService
public void delete(Long videojuegoId, String reviewId, String usuarioActual, boolean esAdmin) {
    // Mismo patrón que update(): cargar (404 si no existe), comprobar que pertenece a videojuegoId
    // (404 si no), comprobar autor o esAdmin (403 si no), y solo entonces reviewRepository.deleteById(reviewId)
    // — el heredado de MongoRepository del que ya habla la teoría, nunca lo habías usado de verdad hasta ahora.
}
```

```java
// En VideojuegoReviewController
@Operation(summary = "Eliminar una reseña (solo el autor o un administrador)")
@ApiResponses({
        @ApiResponse(responseCode = "204", description = "Reseña eliminada correctamente"),
        @ApiResponse(responseCode = "401", description = "No autenticado"),
        @ApiResponse(responseCode = "403", description = "No eres el autor ni un administrador"),
        @ApiResponse(responseCode = "404", description = "Reseña no encontrada")
})
@DeleteMapping("/{reviewId}")
public ResponseEntity<Void> delete(
        @PathVariable Long videojuegoId,
        @PathVariable String reviewId,
        Authentication authentication
) {
    // Misma extracción de esAdmin que en el PUT del Paso 1, y la llamada al service
}
```

Añade también la regla que falta a tu `SecurityConfig`, igual que hiciste con el `PUT`:

```java
.requestMatchers(HttpMethod.DELETE, "/api/v1/videojuegos/*/reviews/*").authenticated()
```

No hace falta repetir las tres pruebas del Paso 2-4 — la lógica de permiso es idéntica y ya la has verificado. Basta con una comprobación:

```bash
curl -i -X DELETE http://localhost:8080/api/v1/videojuegos/1/reviews/{reviewId} \
  -H "Authorization: Bearer $TOKEN_USER"
```

**Comprueba**: `204 No Content`, y que un `GET` posterior a esa reseña ya no la incluye.

**Captura**: la respuesta del `DELETE` con `204 No Content`.

---

## Pregunta de comprensión

¿Por qué la comprobación de autoría devuelve `403 Forbidden` y no `404 Not Found`, sabiendo que la reseña sí existe? ¿Cambiaría tu respuesta si, en vez de una reseña, estuvieras protegiendo un recurso donde no quieres ni confirmar que existe a alguien sin permiso? (No hace falta que cambies el código — solo que razones la diferencia).
