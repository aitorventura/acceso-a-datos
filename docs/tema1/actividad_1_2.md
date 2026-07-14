# 🧪 Actividad 1.2: CRUD completo y DTOs sobre el catálogo

!!! warning "Descarga la plantilla"
    📄 [Plantilla 1.2 — CRUD completo y DTOs sobre el catálogo](plantillas/Actividad_1_2_AD_Plantilla.docx){target="_blank" rel="noopener"}

!!! info "Práctica guiada — con partes a tu cargo"
    Vas a construir, sobre las entidades que creaste en la Actividad 1.1, el CRUD completo de `Videojuego` y de `Estudio`. En la teoría ya tienes un ejemplo completo y funcionando (`Libro`/`Editorial`) — aquí no lo vas a copiar y adaptar el nombre: unos pasos vienen guiados al completo, otros solo con la especificación y pistas, y tienes que escribir tú el código, apoyándote en el patrón que ya conoces.

## Qué vas a practicar

- Definir DTOs de entrada y salida como `record`, distintos de la entidad JPA.
- Construir un controller REST completo (GET, POST, PUT, DELETE) para `Videojuego` y para `Estudio`.
- Aplicar `@Transactional` correctamente en operaciones de escritura y de lectura.
- Gestionar el caso "no encontrado" con `ResponseStatusException`.
- Reconocer qué partes de un patrón ya conocido puedes escribir tú solo, y en cuáles conviene volver a mirar el ejemplo.

---

## Requisitos previos

Tu proyecto de la Actividad 1.1, con las entidades `Estudio` y `Videojuego` ya creadas y verificadas contra la base de datos. Ten a mano la teoría de este apartado — el ejemplo de `Libro`/`Editorial` es la referencia que vas a adaptar en cada paso, no algo que debas releer de memoria.

---

## Paso 1 — Los DTOs, con la especificación (no el código)

!!! warning "Antes de usar `@NotBlank` y compañía: revisa el `pom.xml`"
    Bean Validation (`@NotBlank`, `@NotNull`, `@PositiveOrZero`...) no viene incluido en `spring-boot-starter-webmvc` — es una dependencia aparte. Si al escribir el DTO el IDE no encuentra esas anotaciones para importar, o el proyecto no compila, añade esto al `pom.xml` (detalle en la teoría de este apartado):

    ```xml
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-validation</artifactId>
    </dependency>
    ```

Crea el paquete `catalogo.dto` con dos records — pero esta vez solo tienes la tabla de campos, no el código:

**`VideojuegoCreateDTO`** (lo que se recibe al crear/actualizar):

| Campo | Tipo | Validación |
|---|---|---|
| `titulo` | `String` | `@NotBlank`, `@Size(max = 150)` |
| `precio` | `BigDecimal` | `@NotNull`, `@PositiveOrZero` |
| `fechaLanzamiento` | `LocalDate` | `@NotNull`, `@PastOrPresent` |
| `estudioId` | `Long` | `@NotNull` |

**`VideojuegoResponseDTO`** (lo que se devuelve al consultar):

| Campo | Tipo |
|---|---|
| `id` | `Long` |
| `titulo` | `String` |
| `precio` | `BigDecimal` |
| `fechaLanzamiento` | `LocalDate` |
| `nombreEstudio` | `String` |

Escribe los dos `record`, con el paquete y los `import` que hagan falta (`jakarta.validation.constraints.*`, `java.math.BigDecimal`, `java.time.LocalDate`). Usa `LibroCreateDTO`/`LibroResponseDTO` de la teoría como patrón para la forma general (dónde van los `@NotNull`/`@NotBlank`, cómo se declara un `record` con varias líneas) — pero fíjate en una diferencia real, no cosmética: `LibroResponseDTO` anidaba un `EditorialDTO` completo; aquí `VideojuegoResponseDTO` solo lleva `nombreEstudio`, un `String` suelto.

**Pregunta**: ¿por qué crees que aquí basta con un `String` (`nombreEstudio`) en vez de anidar un `EstudioResponseDTO` completo, como sí hacía `LibroResponseDTO` con `EditorialDTO`? Piensa en qué necesita mostrar realmente un cliente que consulta un videojuego.

**Segunda pregunta**: ¿por qué `VideojuegoResponseDTO` no tiene sentido que lleve anotaciones `@NotBlank`/`@NotNull` como el DTO de creación?

!!! tip "Ese mismo razonamiento se aplica a `Estudio`"
    En el Paso 8 vas a construir el CRUD de `Estudio` completo. Aunque no tiene ninguna relación que resolver, sigue necesitando dos DTOs distintos — piensa por qué antes de llegar allí: ¿qué pasaría si el cliente pudiera mandar un `id` en el `POST` de creación?

---

## Paso 2 — El repository

Esta pieza es mecánica — declárala igual que en la teoría, sin nada que decidir:

```java
package com.tunombre.gamevault.catalogo;

import org.springframework.data.jpa.repository.JpaRepository;

public interface VideojuegoRepository extends JpaRepository<Videojuego, Long> {}
```

```java
package com.tunombre.gamevault.catalogo;

import org.springframework.data.jpa.repository.JpaRepository;

public interface EstudioRepository extends JpaRepository<Estudio, Long> {}
```

Con esto ya tienes `findAll()`, `findById(id)`, `save(entidad)`, `existsById(id)` y `deleteById(id)` disponibles sobre `Videojuego` y `Estudio`, respectivamente.

---

## Paso 3 — La lectura: un método guiado, el otro tuyo

`findById` es el más delicado (hay que resolver el caso "no encontrado"), así que va guiado al completo — junto con `mapToDTO`, que vas a reutilizar en todo lo que queda de actividad:

```java
package com.tunombre.gamevault.catalogo;

import com.tunombre.gamevault.catalogo.dto.VideojuegoResponseDTO;
import lombok.RequiredArgsConstructor;
import org.springframework.http.HttpStatus;
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;
import org.springframework.web.server.ResponseStatusException;

@Service
@RequiredArgsConstructor
public class VideojuegoService {
    private final VideojuegoRepository videojuegoRepository;
    private final EstudioRepository estudioRepository;

    @Transactional(readOnly = true)
    public VideojuegoResponseDTO findById(Long id) {
        return videojuegoRepository.findById(id)
                .map(this::mapToDTO)
                .orElseThrow(() -> new ResponseStatusException(HttpStatus.NOT_FOUND, "Videojuego no encontrado"));
    }

    private VideojuegoResponseDTO mapToDTO(Videojuego v) {
        return new VideojuegoResponseDTO(
                v.getId(), v.getTitulo(), v.getPrecio(), v.getFechaLanzamiento(), v.getEstudio().getNombre()
        );
    }
}
```

**Añade tú `findAll()`** a esta misma clase: devuelve `List<VideojuegoResponseDTO>`, es `@Transactional(readOnly = true)` igual que `findById`, y mapea la lista completa del repository con `mapToDTO` — sin `orElseThrow`, porque una lista vacía no es un error (es exactamente el mismo caso que ya viste con `LibroService.findAll()` en la teoría).

---

## Paso 4 — El controller: un método guiado, el otro tuyo

Mismo reparto que en el service — `getById` guiado, `getAll` para ti:

```java
package com.tunombre.gamevault.catalogo;

import com.tunombre.gamevault.catalogo.dto.VideojuegoResponseDTO;
import lombok.RequiredArgsConstructor;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.*;

@RestController
@RequestMapping("/api/v1/videojuegos")
@RequiredArgsConstructor
public class VideojuegoController {
    private final VideojuegoService videojuegoService;

    @GetMapping("/{id}")
    public ResponseEntity<VideojuegoResponseDTO> getById(@PathVariable Long id) {
        return ResponseEntity.ok(videojuegoService.findById(id));
    }
}
```

**Añade tú `getAll()`**: sin `@PathVariable`, sin argumentos, delega en `videojuegoService.findAll()` y responde `200` con la lista completa.

Arranca tu aplicación y comprueba ambos con `curl` o el navegador antes de seguir — si `getAll()`/`findAll()` no funcionan todavía, no tiene sentido construir la escritura encima.

---

## Paso 5 — `create()`, sin código dado

Aquí ya no hay ningún método de ejemplo — solo la especificación. Añade a `VideojuegoService`:

- Un método `create(VideojuegoCreateDTO dto)` que devuelve `VideojuegoResponseDTO`, anotado `@Transactional` (sin `readOnly`: es una escritura).
- Dentro, en este orden:
    1. Busca el `Estudio` con `estudioRepository.findById(dto.estudioId())`. Si no existe, lanza `ResponseStatusException(HttpStatus.NOT_FOUND, ...)` — mismo patrón que ya usaste en `findById`, aplicado esta vez al estudio, no al videojuego.
    2. Crea un `Videojuego` nuevo (`new Videojuego()`) y rellena sus campos con los setters de Lombok — `titulo`, `precio`, `fechaLanzamiento` desde el DTO, y `estudio` con el objeto que acabas de cargar.
    3. Guarda con `videojuegoRepository.save(...)` y devuelve el resultado pasado por `mapToDTO`.

Y en `VideojuegoController`, un método `create` anotado `@PostMapping`, que recibe `@Valid @RequestBody VideojuegoCreateDTO dto` y responde `201 Created` (`HttpStatus.CREATED`) con el resultado.

Guíate por `LibroService.create()`/`LibroController.create()` de la teoría — es exactamente el mismo patrón, con otros nombres de campo.

Antes de probarlo, hace falta un `Estudio` real al que asociar el videojuego — y el endpoint para crear estudios todavía no existe (lo construyes en el Paso 8). Siémbralo directamente por SQL, desde la misma terminal `psql` que ya usaste en la Actividad 1.1:

```sql
INSERT INTO estudio (nombre, pais) VALUES ('Supergiant Games', 'Estados Unidos');
```

Comprueba con `SELECT * FROM estudio;` que se ha creado con `id = 1` (si es la primera fila que insertas en una tabla recién creada, lo será). Ahora sí, prueba `create()`:

!!! warning "Asegúrate de que el servidor está arrancado"
    Necesitas tu aplicación Spring Boot corriendo (con la terminal del servidor visible) antes de lanzar este `curl` — si la paraste en algún momento, vuelve a arrancarla ahora.

```bash
curl -X POST http://localhost:8080/api/v1/videojuegos \
  -H "Content-Type: application/json" \
  -d '{"titulo":"Hades","precio":24.99,"fechaLanzamiento":"2020-09-17","estudioId":1}'
```

**Predicción**: si en vez de `1` pusieras un `estudioId` que no existe en tu base de datos, ¿qué código de estado esperas recibir? Pruébalo también, cambiando el número, antes de seguir.

---

## Paso 6 — `update()`, el `PUT`

`update()` se parece mucho a `create()`, con dos diferencias: no crea un `Videojuego` nuevo, carga el que ya existe (y responde `404` si no lo encuentra, igual que en `findById`); y usa el mismo `VideojuegoCreateDTO` que `create()` — no hace falta un DTO distinto, los datos que se piden son los mismos.

Escribe `update(Long id, VideojuegoCreateDTO dto)` en `VideojuegoService` (`@Transactional`, sin `readOnly`) y el `@PutMapping("/{id}")` correspondiente en el controller. Si te atascas, `LibroService.update()`/`LibroController.update()` de la teoría siguen exactamente esta misma forma.

---

## Mini-reto — el `DELETE`

Repite el patrón que ya has usado dos veces (cargar/comprobar + actuar). Sin más pistas que estas:

- El método del service debe devolver `void`.
- El endpoint debe responder `204 No Content` si todo va bien.
- Si el `id` no existe, debe comportarse igual que `findById`/`update` ante ese mismo caso.

Complétalo tú mismo en `VideojuegoService` y en `VideojuegoController`.

---

## Paso 7 — Comprobación final de las cuatro operaciones

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

## Paso 8 — `Estudio`: el mismo patrón, completo, ahora sin ejemplo

`Estudio` es más simple que `Videojuego` — no tiene que resolver ninguna relación al crearse o actualizarse (`nombre` y `pais`, nada más). Pero eso no significa que valga un único DTO para todo: sigue habiendo un dato, el `id`, que nunca debe poder mandar el cliente al crear (lo genera la base de datos), y campos de entrada que sí necesitan validación (`@NotBlank` en `nombre`/`pais`) pero que no tiene sentido exigir en la respuesta. Mismo motivo que en `Videojuego`, dos DTOs otra vez. Esta vez no hay ningún fragmento de código dado — tienes la especificación y el patrón que acabas de escribir para `Videojuego`. Construye el CRUD completo, las cuatro operaciones.

**`EstudioCreateDTO`** (lo que se recibe al crear/actualizar), en `catalogo.dto`:

| Campo | Tipo | Validación |
|---|---|---|
| `nombre` | `String` | `@NotBlank` |
| `pais` | `String` | `@NotBlank` |

**`EstudioResponseDTO`** (lo que se devuelve al consultar):

| Campo | Tipo |
|---|---|
| `id` | `Long` |
| `nombre` | `String` |
| `pais` | `String` |

**`EstudioService`**, con el mismo patrón que `VideojuegoService` pero sin `EstudioRepository` adicional que resolver (no hay una segunda entidad relacionada que buscar):

- `findAll()` y `findById(id)` — igual que en `Videojuego`, devolviendo `EstudioResponseDTO` con su propio `mapToDTO(Estudio e)` privado.
- `create(EstudioCreateDTO dto)` — más simple que el de `Videojuego`: no hay ningún `findById` previo, solo `new Estudio()`, rellenar y guardar.
- `update(Long id, EstudioCreateDTO dto)` y `delete(Long id)` — mismo patrón que ya usaste en `VideojuegoService`.

**`EstudioController`**, en `/api/v1/estudios`: las cuatro operaciones — `GET` (los dos), `POST`, `PUT` y `DELETE`, igual que en `VideojuegoController` — incluido el `@Valid` en `create`/`update`, ahora que `EstudioCreateDTO` lleva anotaciones que comprobar.

**Comprueba** con `curl` que las cuatro operaciones de `/api/v1/estudios` funcionan igual que sus equivalentes de `Videojuego`:

```bash
# Crear
curl -X POST http://localhost:8080/api/v1/estudios -H "Content-Type: application/json" \
  -d '{"nombre":"FromSoftware","pais":"Japón"}'

# Leer
curl http://localhost:8080/api/v1/estudios

# Actualizar (sustituye 1 por el id real que te haya devuelto el create)
curl -X PUT http://localhost:8080/api/v1/estudios/1 -H "Content-Type: application/json" \
  -d '{"nombre":"FromSoftware","pais":"Japón"}'

# Borrar
curl -X DELETE http://localhost:8080/api/v1/estudios/1
```

---

## Pregunta final

`create()` tiene una sola operación de guardado (`videojuegoRepository.save(v)`). ¿Qué aporta exactamente `@Transactional` en ese caso, si solo hay una escritura? ¿Y si mañana ese mismo método tuviera que guardar el videojuego *y* actualizar un contador en otra tabla — cambiaría en algo la respuesta?

---

## Entregable

Lo que entregas es la plantilla descargable del principio de la actividad, completa. Como último punto de esa plantilla: una captura del código completo de `EstudioService` o `EstudioController` (el que prefieras), tal y como lo has escrito tú en el Paso 8 — como evidencia de tu propia implementación, no la del ejemplo de la teoría.

---

## ✅ Cierre

Tu GameVault ya tiene un CRUD completo y transaccional sobre `Videojuego` y sobre `Estudio`. Esta vez buena parte del código lo has escrito tú, apoyándote en el patrón de la teoría en vez de copiarlo — es la misma habilidad que vas a necesitar el resto del curso.

Has probado todo esto con `curl`, línea a línea — suficiente para verificar un endpoint suelto, pero incómodo para probar varios seguidos o repetir pruebas. En Programación de Servicios y Procesos, Actividad 1.2, vas a documentar estos mismos endpoints con OpenAPI/Swagger y a probarlos desde una interfaz visual en el navegador — sin dejar de usar `curl` cuando te convenga, pero con una alternativa más cómoda para el día a día.

Todavía lo has hecho todo con Spring Data JPA gestionando la conexión por ti — en la próxima actividad vas a ver, con JDBC puro, toda la "fontanería" que hay debajo de ese `@Transactional` que acabas de usar.
