# 🧪 Actividad 2.1: La columna `detallesPlataforma` — persistiendo objetos estructurados

!!! warning "Descarga la plantilla"
    📄 [Plantilla 2.1 — La columna detallesPlataforma](plantillas/Actividad_2_1_AD_Plantilla.docx){target="_blank" rel="noopener"}

!!! info "Práctica guiada"
    Añades a tu `Videojuego` una columna JSONB real, la expones en tu API, y comprueba con tus propios ojos que PostgreSQL la guarda como JSON de verdad, no como texto plano.

## Qué vas a practicar

- Añadir un campo JSONB a una entidad ya existente.
- Ampliar DTOs para exponer ese campo en la API.
- Verificar en la base de datos que el tipo de columna es realmente `jsonb`.
- Poblar una tabla con datos de prueba a gran escala, y comparar el plan de una consulta antes y después de crear un índice GIN.
- Entender el comportamiento de reemplazo total al actualizar.

---

## Requisitos previos

Tu entidad `Videojuego` y su CRUD completo (Tema 1).

!!! warning "Tu API ya pide credenciales — usa `curl`, no Swagger"
    Si has seguido también Programación de Servicios y Procesos hasta la Actividad 2.3, tu GameVault ya tiene Spring Security activo: el `POST` y el `PUT` de esta actividad necesitan credenciales (`-u admin:admin123`, el mismo usuario que ya tienes sembrado). Swagger UI todavía no es cómodo de usar para esto —no hay JWT ni botón "Authorize" hasta la Actividad 2.4 de PSP—, así que para toda esta actividad usa `curl` con `-u`, tal y como aparece en cada paso.

---

## Paso 1 — El campo en la entidad, y en los DTOs

En `Videojuego.java`:

```java
@JdbcTypeCode(SqlTypes.JSON)
@Column(columnDefinition = "jsonb")
private Map<String, Object> detallesPlataforma;
```

En `VideojuegoCreateDTO` y `VideojuegoResponseDTO`, añade el campo correspondiente:

```java
public record VideojuegoCreateDTO(
        // ... campos ya existentes ...
        Map<String, Object> detallesPlataforma
) {}

public record VideojuegoResponseDTO(
        // ... campos ya existentes ...
        Map<String, Object> detallesPlataforma
) {}
```

Fíjate en que, a diferencia del resto de campos del DTO, `detallesPlataforma` no lleva ninguna anotación de validación. No es un descuido: `@NotNull` estaría mal aquí, porque un videojuego sin esta información (o creado antes de que existiera la columna) queda con `detallesPlataforma = null`, un estado válido. Y validar la forma interna del `Map` —qué claves debe tener, qué tipo debe llevar cada valor— necesitaría un validador a medida, no una anotación estándar de Bean Validation. Es, literalmente, el trade-off de "sin tipado" que ya viste en la teoría, aquí en tu propio código.

Y en el `mapToDTO`/`create`/`update` de tu `VideojuegoService`, asegúrate de que este campo se propaga igual que los demás (`v.setDetallesPlataforma(dto.detallesPlataforma())`, y en el DTO de respuesta).

Reinicia tu aplicación. Con `ddl-auto: update` no hace falta borrar nada: Hibernate detecta el campo nuevo y ejecuta un `ALTER TABLE` que añade la columna a tu tabla `videojuego` ya existente, sin tocar las filas que ya tenías — simplemente quedan con `detalles_plataforma` a `NULL`, un estado válido, no un error.

!!! warning "Esto rompe el test MockMvc que ya tienes de PSP"
    `VideojuegoControllerTest` (Programación de Servicios y Procesos, Actividad 1.3) construye `VideojuegoResponseDTO` con el constructor de los campos antiguos — al añadir `detallesPlataforma` como sexto campo, cualquier `new VideojuegoResponseDTO(...)` con solo cinco argumentos deja de compilar. Hay **tres** sitios donde aparece esa llamada, uno por cada test que construye el DTO a mano — añade `, Map.of()` al final de los tres:

    ```java
    // en getAll_DebeDevolverVideojuegosPaginados()
    var dto = new VideojuegoResponseDTO(1L, "Hades", new BigDecimal("24.99"), LocalDate.of(2020, 9, 17), "Supergiant Games", Map.of());

    // en create_DebeDevolver201_CuandoElDtoEsValido()
    var dto = new VideojuegoResponseDTO(2L, "Celeste", new BigDecimal("19.99"), LocalDate.of(2018, 1, 25), "Extremely OK Games", Map.of());

    // en update_DebeDevolver200_CuandoElDtoEsValido()
    var dto = new VideojuegoResponseDTO(1L, "Hades", new BigDecimal("14.99"), LocalDate.of(2020, 9, 17), "Supergiant Games", Map.of());
    ```

    Y añade el import que falta, junto a los de `BigDecimal`/`LocalDate`/`List`:

    ```java
    import java.util.Map;
    ```

    `Map.of()` es un mapa vacío e inmutable — le basta al test, porque estos casos no comprueban nada sobre `detallesPlataforma` en concreto, solo necesitan que el DTO compile y construya un objeto válido.

---

## Paso 2 — Crear un videojuego con una plataforma

```bash
curl -X POST http://localhost:8080/api/v1/videojuegos \
  -u admin:admin123 \
  -H "Content-Type: application/json" \
  -d '{
    "titulo": "Celeste",
    "precio": 19.99,
    "fechaLanzamiento": "2018-01-25",
    "estudioId": 1,
    "detallesPlataforma": {"steam": {"idApp": 504230, "logros": 45}}
  }'
```

**Comprueba**: que la respuesta incluye `detallesPlataforma` con exactamente esa estructura.

**Anota** el `id` que te devuelve la respuesta — lo necesitarás más adelante, en el Paso 6.

**Captura**: la respuesta completa de este `curl`.

---

## Paso 3 — Varias plataformas anidadas

Sin más indicaciones, crea un segundo videojuego, esta vez con `detallesPlataforma` conteniendo **al menos dos** plataformas distintas, con las claves que tú elijas dentro de cada una (id de la app, número de logros, o lo que consideres relevante). Verifica igual que en el paso anterior.

**Captura**: la respuesta de este segundo `curl`, con las dos plataformas a la vista.

---

## Paso 4 — Verificación en la base de datos

A partir de aquí tienes dos vías para ejecutar SQL contra tu base de datos, igual que en el Tema 1 — usa la que prefieras en este paso y en los siguientes:

- **Herramienta gráfica** (pgAdmin, DBeaver): conecta a `localhost:5432` con las credenciales de tu `docker-compose.yml`.
- **`psql` desde terminal**, como en los comandos de abajo.

```bash
docker exec -it <tu-contenedor-postgres> psql -U gamevault_user -d gamevault_db -c "\d videojuego"
```

!!! note "`\d` es solo de `psql`, no SQL de verdad"
    Si estás en pgAdmin/DBeaver, `\d videojuego` no funciona en el panel de consultas — es un atajo propio del cliente `psql`, no llega al servidor como SQL. Con herramienta gráfica, navega el árbol (`Databases → gamevault_db → Schemas → public → Tables → videojuego → Columns`), o ejecuta esta consulta equivalente, que sí es SQL estándar:
    ```sql
    SELECT column_name, data_type FROM information_schema.columns WHERE table_name = 'videojuego';
    ```
    La misma consulta, si prefieres lanzarla por `psql` en vez de escribir `\d`:
    ```bash
    docker exec -it <tu-contenedor-postgres> psql -U gamevault_user -d gamevault_db \
      -c "SELECT column_name, data_type FROM information_schema.columns WHERE table_name = 'videojuego';"
    ```

**Comprueba** que la columna `detalles_plataforma` (Postgres normaliza a minúsculas) aparece con tipo `jsonb`, no `text` ni `character varying`.

**Captura**: la salida de `\d videojuego` en tu terminal, o la vista de columnas equivalente si has usado pgAdmin/DBeaver — con el tipo de la columna a la vista en cualquiera de los dos casos.

Consulta el contenido directamente en SQL:

```bash
docker exec -it <tu-contenedor-postgres> psql -U gamevault_user -d gamevault_db \
  -c "SELECT titulo, detalles_plataforma FROM videojuego;"
```

**Anota**: ¿el JSON que ves (en `psql` o en pgAdmin/DBeaver) coincide exactamente con el que mandaste por la API?

---

## Paso 5 — Indexar con GIN, antes y después

Con un puñado de filas de prueba nunca vas a ver un índice ganar, por poco que cueste usarlo: recorrer una tabla pequeña entera es más barato que consultar cualquier índice. Antes de crear nada, genera un volumen de datos realista con una sola sentencia:

!!! tip "El volumen no basta — hace falta que la consulta sea selectiva"
    Además de muchas filas, la consulta tiene que devolver **pocas** de ellas en proporción. Si la mitad de la tabla tuviera `"steam"`, seguiría ganando el `Seq Scan` aunque hubiera un millón de filas: recorrer la tabla entera sale más barato que saltar de una en una por el índice cuando te vas a quedar con la mitad de todas formas. Por eso el `INSERT` de abajo reparte `"steam"` en solo **1 de cada 100** filas (`i % 100 = 0`) — una consulta que de verdad descarta la inmensa mayoría de la tabla, el caso donde un índice aporta algo real.

```bash
docker exec -it <tu-contenedor-postgres> psql -U gamevault_user -d gamevault_db -c "
INSERT INTO videojuego (titulo, precio, fecha_lanzamiento, estudio_id, detalles_plataforma)
SELECT
    'Videojuego de prueba ' || i,
    (random() * 60 + 5)::numeric(10,2),
    DATE '2000-01-01' + (i % 9000),
    1,
    CASE WHEN i % 100 = 0
        THEN '{\"steam\": {\"idApp\": 1000}}'::jsonb
        ELSE '{\"switch\": {\"idApp\": 2000}}'::jsonb
    END
FROM generate_series(1, 100000) AS i;
"
```

`generate_series(1, 100000)` genera 100.000 números de golpe, dentro del propio motor, y el `SELECT` construye una fila por cada uno — nada de round-trips fila a fila desde fuera.

!!! tip "Si usas pgAdmin/DBeaver en vez de terminal"
    Pega esto directamente en el panel de consultas — es el mismo `INSERT` de arriba, sin el envoltorio de `docker exec`/`psql` ni las comillas escapadas:
    ```sql
    INSERT INTO videojuego (titulo, precio, fecha_lanzamiento, estudio_id, detalles_plataforma)
    SELECT
        'Videojuego de prueba ' || i,
        (random() * 60 + 5)::numeric(10,2),
        DATE '2000-01-01' + (i % 9000),
        1,
        CASE WHEN i % 100 = 0
            THEN '{"steam": {"idApp": 1000}}'::jsonb
            ELSE '{"switch": {"idApp": 2000}}'::jsonb
        END
    FROM generate_series(1, 100000) AS i;
    ```

Con los datos ya cargados, comprueba el plan **antes** de tener ningún índice (`ANALYZE` actualiza las estadísticas que usa el planificador, para que la comparación sea justa):

```bash
docker exec -it <tu-contenedor-postgres> psql -U gamevault_user -d gamevault_db \
  -c "ANALYZE videojuego; EXPLAIN SELECT * FROM videojuego WHERE detalles_plataforma ? 'steam';"
```

!!! note "El operador `?` lo trabajarás a fondo en el próximo apartado"
    Por ahora, úsalo tal cual, sin más explicación — aquí solo te sirve para tener una consulta real con la que comparar planes. En "Consultas sobre columnas JSONB" verás qué hace exactamente, junto con `jsonb_exists` y cómo integrarlo en una Specification.

**Anota**: con 100.000 filas y sin índice todavía, el plan debería decir `Seq Scan` — es la única opción real que tiene el motor.

Ahora crea el índice:

```bash
docker exec -it <tu-contenedor-postgres> psql -U gamevault_user -d gamevault_db \
  -c "CREATE INDEX idx_videojuego_detalles_plataforma ON videojuego USING GIN (detalles_plataforma);"
```

Y repite exactamente el mismo `EXPLAIN`:

```bash
docker exec -it <tu-contenedor-postgres> psql -U gamevault_user -d gamevault_db \
  -c "EXPLAIN SELECT * FROM videojuego WHERE detalles_plataforma ? 'steam';"
```

**Comprueba**: el plan debería cambiar — en vez de `Seq Scan`, ahora debería aparecer una combinación de `Bitmap Heap Scan` y `Bitmap Index Scan` sobre `idx_videojuego_detalles_plataforma`. Con 100.000 filas de por medio, esta vez la diferencia no es cuestión de suerte.

!!! example "Cómo leer y comparar los dos planes"
    Cada línea de `EXPLAIN` trae `cost=X..Y`: `X` es el coste de arrancar (devolver la primera fila), `Y` es el coste **total** estimado para terminar la consulta. Para comparar planes, el número que importa es `Y`.

    **Antes** (una sola línea): `Seq Scan on videojuego (cost=0.00..4699.19 rows=997 ...)` — recorre la tabla entera, fila a fila, comprobando el filtro en cada una. Coste total: `4699.19`.

    **Después** (dos líneas anidadas, de dentro hacia fuera):
    ```
    Bitmap Heap Scan on videojuego (cost=26.13..2211.22 rows=997 ...)
      Recheck Cond: (detalles_plataforma ? 'steam'::text)
      ->  Bitmap Index Scan on idx_videojuego_detalles_plataforma (cost=0.00..25.88 rows=997 ...)
            Index Cond: (detalles_plataforma ? 'steam'::text)
    ```
    Ahora son **dos pasos**, no uno. Primero, el `Bitmap Index Scan` (la línea de dentro, con la flecha `->`) consulta tu índice GIN para averiguar **dónde** están las filas que cumplen la condición, sin tocar la tabla todavía — coste `25.88`, prácticamente gratis, porque el índice ya sabe la respuesta. Después, el `Bitmap Heap Scan` (la línea de fuera, que envuelve a la anterior) usa esa lista de ubicaciones para ir **directamente** a esas filas concretas de la tabla, sin mirar el resto — coste total (ya incluye el paso anterior): `2211.22`.

    La comparación que importa es `4699.19` (antes) frente a `2211.22` (después): la consulta pasa a costar menos de la mitad. En vez de mirar 100.000 filas para descartar el 99%, el motor va directo a las ~1.000 que sí importan.

**Captura**: los dos planes, el de antes y el de después, uno junto al otro — desde tu terminal con `psql`, o desde la pestaña "Explain" si usas pgAdmin/DBeaver.

---

## Paso 6 — Actualizar y observar el reemplazo total

Actualiza tu "Celeste" del Paso 2 con un `PUT`, cambiando `detallesPlataforma` a una estructura **distinta** (por ejemplo, solo con la clave `"switch"`, sin `"steam"`). Sustituye `<id-de-celeste>` por el `id` que anotaste en el Paso 2 — **no** des por hecho que es `1`: si ya tenías videojuegos de temas anteriores, el tuyo tendrá otro número:

```bash
curl -X PUT http://localhost:8080/api/v1/videojuegos/<id-de-celeste> \
  -u admin:admin123 \
  -H "Content-Type: application/json" \
  -d '{
    "titulo": "Celeste",
    "precio": 19.99,
    "fechaLanzamiento": "2018-01-25",
    "estudioId": 1,
    "detallesPlataforma": {"switch": {"idApp": "ABCD"}}
  }'
```

**Comprueba** consultando de nuevo — ahora con la tabla llena de las 100.000 filas de prueba del Paso 5, filtra explícitamente por este videojuego en vez de listar la tabla entera (de nuevo, con tu `id` real, no necesariamente `1`):

```bash
docker exec -it <tu-contenedor-postgres> psql -U gamevault_user -d gamevault_db \
  -c "SELECT titulo, detalles_plataforma FROM videojuego WHERE id = <id-de-celeste>;"
```

¿Sigue estando `"steam"` en el JSON guardado, o ha desaparecido por completo?

**Captura**: el resultado de esta consulta, ya con `detallesPlataforma` actualizado — desde `psql` o desde pgAdmin/DBeaver, la que hayas usado.

**Pregunta de comprensión**: ¿por qué el `PUT` reemplaza el objeto JSON completo en vez de combinar (*merge*) el nuevo contenido con el anterior? Relaciona tu respuesta con cómo funciona el mapeo `Map` → `jsonb`: ¿en qué punto del proceso Hibernate tendría que decidir "combinar" en vez de "sustituir", y por qué no lo hace por defecto?

---

## Pregunta final

¿Qué ventaja concreta tiene JSONB frente a crear una tabla `plataforma_videojuego` (con una fila por plataforma y sus columnas propias)? ¿En qué situación sería mejor la tabla relacional en vez de JSONB — piensa en un caso donde necesitaras, por ejemplo, buscar todos los videojuegos disponibles en una plataforma concreta de forma muy eficiente, o donde la estructura de cada plataforma tuviera que cumplir reglas estrictas?

---

## ✅ Cierre

Tu `Videojuego` ya persiste objetos estructurados reales en una columna JSONB. Todavía no has consultado por su contenido — solo lo has guardado y leído entero. En la próxima actividad vas a filtrar videojuegos según qué plataformas tienen, usando `jsonb_exists`.

!!! tip "¿Quieres quitar las 100.000 filas de prueba del Paso 5?"
    Se quedan en tu base de datos de desarrollo sin causar ningún problema, pero si prefieres limpiarlas: si sigues teniendo un `data.sql` que reinicializa tus tablas (PSP, Actividad 2.3), pon temporalmente `spring.sql.init.mode` de nuevo en `always` en tu `application-dev.yaml` y reinicia — se ejecutará y dejará la base de datos como la tenías. Vuelve a ponerlo en `never` después, o volverás a perder cualquier dato nuevo en cada arranque, que es justo el problema que evitaste en su momento.
