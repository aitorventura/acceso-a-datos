# 🧪 Actividad 1.4: Procedimiento almacenado con `JdbcTemplate`

!!! warning "Descarga la plantilla"
    📄 [Plantilla 1.4 — Procedimiento almacenado con JdbcTemplate](plantillas/Actividad_1_4_AD_Plantilla.docx){target="_blank" rel="noopener"}

!!! info "Con la teoría delante, no para copiar y pegar"
    Hoy añades a tu GameVault el procedimiento almacenado `ajustar_precio_estudio`, lo invocas desde Java con `JdbcTemplate` y lo expones como un endpoint REST nuevo — usando como referencia el `ajustar_precio_editorial` de la teoría, sin copiarlo directamente: tu tabla, tus columnas y tu dominio son distintos.

## Qué vas a practicar

- Escribir un procedimiento almacenado en PostgreSQL, adaptando un patrón visto en la teoría.
- Invocarlo desde Spring con `JdbcTemplate`.
- Exponerlo como un endpoint REST siguiendo el estilo ya establecido del proyecto.
- Declarar tus propias consultas derivadas por nombre en un repository, y exponerlas como endpoints nuevos.

---

## Requisitos previos

Tu CRUD completo de `Videojuego` y de `Estudio` de la Actividad 1.2, algún dato de prueba (un estudio con varios videojuegos asociados), y haber leído la teoría de este apartado — en concreto el ejemplo completo `ajustar_precio_editorial`/`EditorialService`, que vas a adaptar paso a paso.

---

## Paso 1 — El procedimiento almacenado, con la teoría delante

En la teoría de este apartado tienes `ajustar_precio_editorial`: un procedimiento que sube o baja el precio de todos los libros de una editorial, en un porcentaje. Escribe tú mismo el equivalente para tu catálogo:

```sql
CREATE OR REPLACE PROCEDURE ajustar_precio_estudio(p_estudio_id BIGINT, p_porcentaje NUMERIC)
LANGUAGE plpgsql
AS $$
BEGIN
    -- tu UPDATE aquí
END;
$$;
```

No es un procedimiento distinto, es el mismo patrón sobre otra tabla: `videojuego` en vez de `libro`, `estudio_id` en vez de `editorial_id`. Fíjate especialmente en si sigue haciendo falta el `ROUND(..., 2)` de la teoría — piensa por qué estaba ahí.

!!! warning "Ejecuta esto sobre la base de datos de tu proyecto, no en otra"
    Este `CREATE OR REPLACE PROCEDURE` no lo ejecuta tu aplicación Java — lo tienes que lanzar tú directamente contra PostgreSQL, igual que cualquier otra sentencia SQL: desde el terminal de `psql` (dentro del Dev Container, o conectándote a él desde fuera), o desde una interfaz gráfica como pgAdmin o DBeaver. Sea cual sea la vía, tiene que ser contra la misma base de datos a la que se conecta tu aplicación Spring Boot — la del Dev Container de GameVault, no una instancia ni una base de datos distinta que tengas por otro lado. Un procedimiento almacenado vive dentro de una base de datos concreta: si lo creas en otra, `JdbcTemplate` fallará al invocarlo desde tu service porque, para esa base de datos, el procedimiento simplemente no existe.

**Predicción**: antes de invocarlo, anota en un papel qué precio final esperas para un videojuego que cuesta `20.00` si aplicas un porcentaje de `10`. Ejecuta tu `CREATE OR REPLACE PROCEDURE` (por terminal o interfaz gráfica, como acabas de ver) y comprueba si tu predicción ha coincidido:

```sql
SELECT id, titulo, precio FROM videojuego WHERE estudio_id = 1;
CALL ajustar_precio_estudio(1, 10); -- sube un 10%
SELECT id, titulo, precio FROM videojuego WHERE estudio_id = 1;
```

**Captura**: los dos `SELECT`, antes y después de la llamada, con el precio ya cambiado.

---

## Paso 2 — Invocación desde el service, sin código dado

Añade `JdbcTemplate` a `EstudioService`, junto al repositorio que ya tienes — se inyecta exactamente igual que `EstudioRepository`, con `@RequiredArgsConstructor`. No necesitas registrarlo en ningún sitio: Spring Boot lo configura automáticamente en cuanto detecta un `DataSource` en el classpath (el mismo que ya usa tu Spring Data JPA desde la Actividad 1.1).

Con eso, y mirando el patrón de `EditorialService.ajustarPrecio` de la teoría, escribe tú mismo:

- Un DTO `AjustePrecioDTO` con un único campo `porcentaje` de tipo `BigDecimal`, validado para que no llegue vacío y para que no tenga más cifras de las que admite la columna `precio` (`10,2` en la base de datos: hasta 8 dígitos enteros y 2 decimales). Busca en `jakarta.validation.constraints` qué anotación limita el número de dígitos de un `BigDecimal`.
- Un método `ajustarPrecio(Long estudioId, BigDecimal porcentaje)` en `EstudioService`, anotado `@Transactional` (igual que `update`/`delete`, porque también combina una lectura con una escritura), que compruebe primero que el estudio existe y después invoque el procedimiento con `jdbcTemplate.update(...)`.
- Un endpoint `POST /api/v1/estudios/{id}/ajustar-precio` en `EstudioController`, con `@Valid` sobre el DTO — el mismo patrón que ya usas en el resto de endpoints de escritura. Documéntalo igual que a todos los demás: este endpoint concreto no es opcional ni un "extra" si te sobra tiempo, tiene que llevar `@Operation`/`@ApiResponses` igual que cada endpoint que ya documentaste en la Actividad 1.2 de PSP, con estos códigos:

```java
@ApiResponses({
        @ApiResponse(responseCode = "204", description = "..."),
        @ApiResponse(responseCode = "400", description = "..."),
        @ApiResponse(responseCode = "404", description = "...")
})
```

Prueba con `curl` o Swagger UI:

```bash
curl -X POST http://localhost:8080/api/v1/estudios/1/ajustar-precio \
  -H "Content-Type: application/json" \
  -d '{"porcentaje": -15}'
```

**Comprueba**: con un `GET /api/v1/videojuegos` (o consultando directamente en `psql`), que los precios de los videojuegos de ese estudio han bajado un 15%.

**Captura**: la petición y su respuesta, junto con el `GET` posterior que confirma el cambio de precio.

**Pregunta**: compara tu `AjustePrecioDTO`, tu `EstudioService.ajustarPrecio` y tu endpoint con `ajustar_precio_editorial`/`EditorialService` de la teoría. ¿Qué has tenido que cambiar y qué se mantiene exactamente igual?

**Pregunta**: prueba a enviar el DTO con `porcentaje` vacío, y después con un valor de más de 8 dígitos enteros (por ejemplo `123456789`). ¿Qué código de estado obtienes en cada caso? ¿Por qué esos dos casos concretos disparan la validación y no otros?

!!! note "Esta actividad no pide tests para este endpoint"
    No forma parte de esta actividad, pero en un proyecto real lo ideal sería completar `ajustarPrecio` con un test de controller (con MockMvc) que compruebe tanto el caso correcto como los dos casos de validación fallida que acabas de probar a mano — el mismo tipo de test que trabajarás en profundidad en Programación de Servicios y Procesos.

---

## Mini-reto — una salvaguarda contra precios negativos

Ahora mismo, si aplicas un porcentaje muy negativo (por ejemplo, `-150`), el procedimiento dejaría precios en negativo — algo que no tiene sentido de negocio. Modifica el `UPDATE` del procedimiento para que ningún precio resultante quede por debajo de `0`.

**Pista**: la función SQL `GREATEST(a, b)` devuelve el mayor de los dos valores — puedes envolver el cálculo del nuevo precio con ella para que nunca baje de `0`.

**Predicción**: antes de corregirlo, anota qué precio esperas ver si aplicas ahora mismo un `-150` sobre un videojuego de `20.00` sin la salvaguarda.

Vuelve a ejecutar `CREATE OR REPLACE PROCEDURE` con tu versión corregida (el `OR REPLACE` te permite redefinirlo sin borrarlo antes) y comprueba con un porcentaje muy negativo que ahora los precios se quedan en `0`, no en negativo.

**Captura**: el resultado del `SELECT` tras aplicar el porcentaje muy negativo, con los precios en `0`, no en negativo.

---

## Paso 3 — Consultas derivadas, sobre tu propio catálogo

Ya viste en la teoría de este apartado la convención completa de consultas derivadas por nombre. Añade a `VideojuegoRepository` (una interfaz vacía hasta ahora) estos dos métodos, sin cuerpo:

```java
List<Videojuego> findByEstudioIdOrderByPrecioDesc(Long estudioId);
List<Videojuego> findByTituloContainingIgnoreCase(String fragmento);
```

Expón cada uno con un endpoint `GET` nuevo en `VideojuegoController`, siguiendo el mismo patrón que ya conoces (service intermedio, `mapToDTO`, `ResponseEntity.ok(...)`):

- `GET /api/v1/videojuegos/por-estudio/{estudioId}` — usa `findByEstudioIdOrderByPrecioDesc`, con `@PathVariable`.
- `GET /api/v1/videojuegos/buscar?titulo=...` — usa `findByTituloContainingIgnoreCase`. Aquí el dato no identifica un recurso (no encaja como `@PathVariable`), es un filtro opcional sobre una búsqueda — para eso está `@RequestParam`, que captura un parámetro de la *query string* de la URL: `@RequestParam String titulo` recoge el valor de `?titulo=...`.

No hay código dado para el service ni el controller — ya sabes escribir ese patrón desde la Actividad 1.2.

!!! warning "El orden de las rutas importa"
    Declara `@GetMapping("/buscar")` y `@GetMapping("/por-estudio/{estudioId}")` **antes** que `@GetMapping("/{id}")` en la clase — si no, Spring intenta interpretar `buscar` como un `id` y falla al convertirlo a `Long`.

**Comprueba** con `curl` ambos endpoints contra datos reales de tu proyecto.

**Captura**: la respuesta de ambos endpoints (`/por-estudio/{estudioId}` y `/buscar?titulo=...`) con datos reales de tu catálogo.

!!! note "Tampoco se piden tests para estos dos endpoints"
    Igual que con `ajustarPrecio` en el Paso 2: no forma parte de esta actividad, pero en un proyecto real lo ideal sería completarlos con tests de controller.

**Pregunta**: si `findByTituloContainingIgnoreCase("celeste")` no encontrara nada aunque exista un videojuego llamado "Celeste", ¿qué comprobarías primero? El nombre del método ya cubre mayúsculas/minúsculas — piensa en qué otra cosa, más básica, podría estar fallando.

---

## ✅ Cierre

En el Tema 2 vuelves a la comodidad de Spring Data JPA, pero esta vez con Hibernate como herramienta ORM completa — vas a instalarla, configurarla y usarla para persistir objetos sin escribir SQL a mano, retomando exactamente el desfase objeto-relacional del que arrancó este tema.
