# 🧪 Actividad 2.2: Filtrar por plataforma con `jsonb_exists`

!!! warning "Descarga la plantilla"
    📄 [Plantilla 2.2 — Filtrar por plataforma con jsonb_exists](plantillas/Actividad_2_2_AD_Plantilla.docx){target="_blank" rel="noopener"}

!!! info "Práctica guiada"
    Añades el filtro por plataforma a tu listado de videojuegos, usando `jsonb_exists` sobre la columna JSONB de la Actividad 2.1 — y, más adelante, filtrando también por un valor anidado dentro de ella.

## Qué vas a practicar

- Invocar una función SQL nativa desde Criteria API con `criteriaBuilder.function(...)`.
- Combinar un filtro JSONB con los filtros ya existentes.
- Verificar el comportamiento de modificación de un campo JSONB filtrado.
- Filtrar por un valor anidado dentro del JSON, con `jsonb_extract_path_text`.
- Repetir los dos filtros JSONB (existencia y valor anidado) con `@Query(nativeQuery = true)`, y saber cuándo conviene una u otra vía.

---

## Requisitos previos

Tu columna `detallesPlataforma` de la Actividad 2.1, con videojuegos de distintas plataformas ya creados; tus Specifications de la Actividad 1.5.

### Datos de prueba con plataformas ya cargados

Tu `data.sql` (Tema 1) todavía no rellena `detalles_plataforma` — esa columna no existía cuando lo has escrito. Actualízalo, añadiendo esa columna a cada `INSERT` con Steam y, en varios casos, también Switch: así puedes probar todos los pasos de esta actividad sin tener que crear datos a mano primero. Este es el fichero completo, listo para copiar tal cual:

```sql
TRUNCATE TABLE videojuego, estudio RESTART IDENTITY CASCADE;

INSERT INTO estudio (nombre, pais) VALUES
                                       ('Supergiant Games', 'Estados Unidos'),
                                       ('Team Cherry', 'Australia'),
                                       ('Motion Twin', 'Francia'),
                                       ('Toby Fox', 'Estados Unidos'),
                                       ('Klei Entertainment', 'Canadá'),
                                       ('ConcernedApe', 'Estados Unidos'),
                                       ('Innersloth', 'Estados Unidos');

INSERT INTO videojuego (titulo, precio, fecha_lanzamiento, estudio_id, detalles_plataforma) VALUES
    ('Hades', 24.99, '2020-09-17', 1, '{"steam": {"idApp": 1145360}, "switch": {"idApp": "HDS01"}}'::jsonb),
    ('Pyre', 19.99, '2017-07-25', 1, '{"steam": {"idApp": 462770}}'::jsonb),
    ('Transistor', 19.99, '2014-05-20', 1, '{"steam": {"idApp": 237930}, "switch": {"idApp": "TRS01"}}'::jsonb),
    ('Bastion', 14.99, '2011-07-20', 1, '{"steam": {"idApp": 107100}}'::jsonb),
    ('Hades II', 29.99, '2024-05-06', 1, '{"steam": {"idApp": 1145350}}'::jsonb),
    ('Hollow Knight', 14.99, '2017-02-24', 2, '{"steam": {"idApp": 367520}, "switch": {"idApp": "HK001"}}'::jsonb),
    ('Hollow Knight: Silksong', 19.99, '2025-09-04', 2, '{"steam": {"idApp": 1030300}, "switch": {"idApp": "HKS01"}}'::jsonb),
    ('Dead Cells', 24.99, '2018-08-07', 3, '{"steam": {"idApp": 588650}, "switch": {"idApp": "DC001"}}'::jsonb),
    ('Undertale', 9.99, '2015-09-15', 4, '{"steam": {"idApp": 391540}}'::jsonb),
    ('Deltarune', 0.00, '2018-10-31', 4, '{"steam": {"idApp": 1671210}, "switch": {"idApp": "DTR01"}}'::jsonb),
    ('Mark of the Ninja', 14.99, '2012-09-07', 5, '{"steam": {"idApp": 214560}}'::jsonb),
    ('Don''t Starve', 14.99, '2013-04-23', 5, '{"steam": {"idApp": 219740}, "switch": {"idApp": "DS001"}}'::jsonb),
    ('Don''t Starve Together', 14.99, '2016-04-21', 5, '{"steam": {"idApp": 322330}, "switch": {"idApp": "DST01"}}'::jsonb),
    ('Oxygen Not Included', 24.99, '2019-07-30', 5, '{"steam": {"idApp": 457140}}'::jsonb),
    ('Invisible, Inc.', 19.99, '2015-05-12', 5, '{"steam": {"idApp": 243970}}'::jsonb),
    ('Griftlands', 24.99, '2021-10-12', 5, '{"steam": {"idApp": 601840}, "switch": {"idApp": "GRF01"}}'::jsonb),
    ('Stardew Valley', 13.99, '2016-02-26', 6, '{"steam": {"idApp": 413150}, "switch": {"idApp": "STV01"}}'::jsonb),
    ('Among Us', 4.99, '2018-06-15', 7, '{"steam": {"idApp": 945360}, "switch": {"idApp": "AU001"}}'::jsonb);
```

!!! tip "Actívalo con `spring.sql.init.mode: always`"
    Este `data.sql` solo se ejecuta si `spring.sql.init.mode` está en `always` en tu `application-dev.yaml` — si ya lo has puesto en `never` (Actividad 2.3 de PSP, para no perder los usuarios que creas a mano), ponlo temporalmente de nuevo en `always`, reinicia para cargar estos datos, y vuelve a dejarlo en `never` después — si no, perderás cualquier dato nuevo en cada arranque, el mismo problema que ya has evitado antes.

!!! warning "Tu API ahora usa JWT, no `-u usuario:contraseña`"
    Si has seguido también Programación de Servicios y Procesos hasta la Actividad 2.4, tu GameVault ya ha sustituido HTTP Basic por JWT: las peticiones que modifican datos necesitan un token (`Authorization: Bearer <token>`), no la cabecera `-u` que has usado en la Actividad 2.1. El Paso 4 de esta actividad te recuerda cómo conseguirlo.

    A diferencia de la Actividad 2.1 —donde Swagger UI todavía no era cómodo de usar—, ahora sí tiene un botón "Authorize" real (PSP, Actividad 2.4): si prefieres no repetir `-H "Authorization: Bearer $TOKEN"` en cada `curl`, pega el token de `admin` una sola vez en Swagger (`/documentacion`) y prueba los pasos de esta actividad con "Try it out" en su lugar. Los ejemplos de aquí siguen en `curl` porque son más fáciles de copiar y pegar tal cual, pero cualquiera de las dos formas es válida.

---

## Paso 1 — La Specification, guiada al completo

En `VideojuegoSpecifications`:

```java
public static Specification<Videojuego> disponibleEnPlataforma(String plataforma) {
    if (plataforma == null || plataforma.isBlank()) {
        return Specification.unrestricted();
    }

    return (root, query, criteriaBuilder) ->
            criteriaBuilder.isTrue(
                    criteriaBuilder.function(
                            "jsonb_exists",
                            Boolean.class,
                            root.get("detallesPlataforma"),
                            criteriaBuilder.literal(plataforma.toLowerCase())
                    )
            );
}
```

Por qué cada pieza: `criteriaBuilder.function("jsonb_exists", Boolean.class, ...)` invoca esa función nativa de PostgreSQL, indicando que devuelve un `Boolean`; `root.get("detallesPlataforma")` es la columna sobre la que se aplica; `criteriaBuilder.literal(plataforma.toLowerCase())` es la clave a buscar, pasada como literal (no como columna) — y en minúsculas, para que la búsqueda no dependa de cómo escribió el usuario la plataforma (asumiendo que también guardas las claves del JSON en minúsculas, como en tus datos de la Actividad 2.1). `criteriaBuilder.isTrue(...)` envuelve el resultado booleano de la función para usarlo como condición del `WHERE`.

---

## Paso 2 — Añadir el filtro al DTO y combinarlo

Añade el campo `plataforma` a tu `VideojuegoFiltroDTO`:

```java
public record VideojuegoFiltroDTO(
        String titulo,
        BigDecimal precioMin,
        BigDecimal precioMax,
        Long estudioId,
        String plataforma
) {}
```

Ahora, sin más indicaciones, encadena `disponibleEnPlataforma(filtro.plataforma())` con `.and(...)` en `findAllPaginated()` de tu `VideojuegoService` — es exactamente el mismo patrón que ya has repetido varias veces en la Actividad 1.5, así que no necesitas código nuevo mostrado: solo añadir una línea más a la cadena que ya tienes.

---

## Paso 3 — Prueba con peticiones reales

```bash
# Plataforma que SÍ existe en tus datos de prueba
curl "http://localhost:8080/api/v1/videojuegos?plataforma=steam"

# Plataforma que NO existe en ningún videojuego
curl "http://localhost:8080/api/v1/videojuegos?plataforma=xbox"
```

**Comprueba**: que el primer caso devuelve solo los videojuegos que de verdad tienen la clave `"steam"` en su `detallesPlataforma`, y el segundo devuelve una lista vacía (no un error).

**Captura**: las dos respuestas, una junto a la otra.

---

## Paso 4 — Modificar y comprobar que el filtro reacciona

Consigue primero un token — el `PUT` que viene ahora ya no acepta `-u usuario:contraseña`, como has visto en el aviso de "Requisitos previos":

```bash
curl -s -X POST http://localhost:8080/api/v1/auth/login \
  -H "Content-Type: application/json" \
  -d '{"username":"admin","password":"admin123"}'
```

Copia el `accessToken` de la respuesta:

```bash
TOKEN="pega-aqui-el-token"
```

Actualiza "Hades" —el primer videojuego que carga tu `data.sql`, así que si lo has recargado hace poco (más arriba, en "Requisitos previos"), tendrá `id = 1`; si no estás seguro, compruébalo primero con un `GET`—, quitando la clave `"steam"` de su `detallesPlataforma`:

```bash
curl -X PUT http://localhost:8080/api/v1/videojuegos/1 \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "titulo": "Hades",
    "precio": 24.99,
    "fechaLanzamiento": "2020-09-17",
    "estudioId": 1,
    "detallesPlataforma": {"switch": {"idApp": "ABCD"}}
  }'
```

Repite el filtro por `steam`:

```bash
curl "http://localhost:8080/api/v1/videojuegos?plataforma=steam"
```

**Comprueba**: que ese videojuego ya no aparece en el resultado, porque su `detallesPlataforma` ya no contiene la clave `"steam"` (recuerda: el `PUT` reemplaza el objeto completo, como has visto en la Actividad 2.1).

**Captura**: la respuesta del `PUT`, y el listado filtrado por `steam` ya sin ese videojuego, una junto a la otra.

!!! warning "Antes de seguir: vuelve a poner `spring.sql.init.mode` en `never`"
    Si sigues con `always` activo desde "Requisitos previos", el próximo reinicio de la aplicación —por cualquier motivo, no hace falta que lo hagas tú a propósito— vuelve a ejecutar `data.sql`, con su `TRUNCATE`: el `PUT` que acabas de hacer desaparece, "Hades" recupera su `detallesPlataforma` original (con `"steam"` otra vez, y el `idApp` de Switch vuelto a `"HDS01"`), y el Paso 5 ya no encontrará nada con `idApp = "ABCD"`. Pon `spring.sql.init.mode` de vuelta en `never` ahora, antes de continuar.

---

## Paso 5 — Filtrar por un valor anidado: `idApp` exacto

`jsonb_exists` solo te dice si una plataforma está presente — no mira sus datos internos. Ahora ve un paso más allá, con la técnica de rutas anidadas del apartado 2 de la teoría (`jsonb_extract_path_text`): encuentra el videojuego cuyo `idApp` de Switch es exactamente `"ABCD"` — el que acabas de dejar en el Paso 4.

Añade a `VideojuegoSpecifications` un método `conIdAppSwitch(String idApp)`, con el mismo patrón que `disponibleEnPlataforma` del Paso 1, pero con `jsonb_extract_path_text` en vez de `jsonb_exists`, y comparando con `criteriaBuilder.equal(...)` en vez de envolver el resultado en `criteriaBuilder.isTrue(...)` — esta función devuelve un `String`, no un `Boolean`. La ruta que tienes que navegar es `detallesPlataforma -> 'switch' -> 'idApp'`; en `jsonb_extract_path_text`, cada nivel de la ruta es un argumento más, en el mismo orden.

Añade el campo `idAppSwitch` a tu `VideojuegoFiltroDTO` y encadénalo con `.and(...)`, igual que has hecho en el Paso 2 con `plataforma`.

Pruébalo:

```bash
curl "http://localhost:8080/api/v1/videojuegos?idAppSwitch=ABCD"
```

**Comprueba**: que el único resultado es el videojuego que has actualizado en el Paso 4 — ni los que tienen Switch con otro `idApp`, ni los que no tienen Switch en absoluto.

**Captura**: esta respuesta.

---

## Paso 6 — Las mismas dos preguntas, con `@Query` nativo

Retoma el Tema 1: cuando JPQL no llegaba a algo, has usado `@Query(nativeQuery = true)`. `jsonb_exists`, `?`, `->`/`->>` tampoco existen en JPQL — es el mismo caso. Vas a repetir, en SQL nativo, las dos preguntas que ya has resuelto con Specifications: "¿existe esta plataforma?" (Paso 1) y "¿coincide un valor de dentro de otra clave?" (Paso 5). Lo único que tienes que escribir tú es la consulta del repositorio — el service y el controller ya están hechos.

!!! note "Sí, esto duplica una consulta que ya tenías — es a propósito"
    En un proyecto real no tendría mucho sentido mantener dos endpoints distintos que devuelven exactamente lo mismo: escogerías una sola vía y te quedarías con ella (probablemente la Specification, precisamente porque se combina con el resto de filtros). Aquí lo repites con fines pedagógicos, para ver con tus propios datos las dos formas de llegar al mismo resultado, y poder comparar en la práctica sus diferencias —sintaxis, tipo de retorno, si se puede combinar con `.and(...)`— en vez de solo leerlas en la teoría.

### 6a — Existencia: ¿tiene esta plataforma?

Añade a `VideojuegoRepository` este método, **escribiendo tú el `@Query`** que falta encima:

```java
// TODO: @Query(value = "...", nativeQuery = true)
// Usa jsonb_exists(...), no el operador ? — choca con el marcador de parámetro de JDBC.
List<Videojuego> buscarPorPlataformaNativo(@Param("plataforma") String plataforma);
```

El service y el controller ya están construidos, tal cual los vas a usar:

```java
// VideojuegoService
@Transactional(readOnly = true)
public List<VideojuegoResponseDTO> buscarPorPlataformaNativo(String plataforma) {
    return videojuegoRepository.buscarPorPlataformaNativo(plataforma).stream()
            .map(this::mapToDTO)
            .toList();
}
```

```java
// VideojuegoController
@Operation(summary = "Filtrar videojuegos por plataforma, con SQL nativo (jsonb_exists)")
@ApiResponses({
        @ApiResponse(responseCode = "200", description = "Videojuegos que tienen esa plataforma en detallesPlataforma")
})
@GetMapping("/por-plataforma-nativa")
public ResponseEntity<List<VideojuegoResponseDTO>> getPorPlataformaNativa(@RequestParam String plataforma) {
    return ResponseEntity.ok(videojuegoService.buscarPorPlataformaNativo(plataforma));
}
```

Pruébalo y compáralo con `?plataforma=steam` del Paso 3:

```bash
curl "http://localhost:8080/api/v1/videojuegos/por-plataforma-nativa?plataforma=steam"
```

**Comprueba**: que devuelve los mismos videojuegos que el Paso 3, con una diferencia esperada: "Hades" ya no aparece, porque en el Paso 4 le quitaste la clave `"steam"` — si sigue apareciendo, algo no se ha actualizado como pensabas.

**Captura**: la respuesta de este endpoint, junto a la del Paso 3.

### 6b — Valor anidado: ¿coincide el `idApp` de dentro de una plataforma?

Mismo patrón, otro método en `VideojuegoRepository` — **escribe tú el `@Query`**:

```java
// TODO: @Query(value = "...", nativeQuery = true)
// Navega detallesPlataforma -> 'switch' ->> 'idApp'. Aquí no hay conflicto con ?,
// así que no hace falta ninguna función: escribe la ruta tal cual, como en psql.
List<Videojuego> buscarPorIdAppSwitchNativo(@Param("idApp") String idApp);
```

Service y controller, ya hechos:

```java
// VideojuegoService
@Transactional(readOnly = true)
public List<VideojuegoResponseDTO> buscarPorIdAppSwitchNativo(String idApp) {
    return videojuegoRepository.buscarPorIdAppSwitchNativo(idApp).stream()
            .map(this::mapToDTO)
            .toList();
}
```

```java
// VideojuegoController
@Operation(summary = "Filtrar videojuegos por idApp de Switch, con SQL nativo (ruta anidada)")
@ApiResponses({
        @ApiResponse(responseCode = "200", description = "Videojuegos cuyo detallesPlataforma.switch.idApp coincide exactamente")
})
@GetMapping("/por-idapp-switch-nativo")
public ResponseEntity<List<VideojuegoResponseDTO>> getPorIdAppSwitchNativo(@RequestParam String idApp) {
    return ResponseEntity.ok(videojuegoService.buscarPorIdAppSwitchNativo(idApp));
}
```

Pruébalo y compáralo con `?idAppSwitch=ABCD` del Paso 5:

```bash
curl "http://localhost:8080/api/v1/videojuegos/por-idapp-switch-nativo?idApp=ABCD"
```

**Comprueba**: que devuelve exactamente el mismo videojuego que el Paso 5 — el que has actualizado en el Paso 4.

**Captura**: la respuesta de este endpoint, junto a la del Paso 5.

### Pregunta

Ninguna de las dos consultas de este paso se puede combinar con `.and(...)` como `disponibleEnPlataforma` o `conIdAppSwitch`. Si quisieras filtrar por plataforma **y** por precio mínimo a la vez, ¿por qué `@Query` no sirve aquí y sí una Specification? Relaciónalo con lo que ya has visto en el Tema 1 sobre cuándo usar cada una de las tres vías de consulta (naming de método, Specifications, `@Query`).

---

## Pregunta final

¿Qué haría falta para filtrar por "disponible en Steam **Y** en Switch a la vez"? Piensa en dos enfoques posibles: encadenar dos veces `disponibleEnPlataforma` con `.and(...)` (una vez por cada plataforma) frente a usar una función distinta de PostgreSQL como `jsonb_exists_all` (que comprueba varias claves de golpe). No hace falta que implementes ninguno de los dos — explica con tus palabras qué diferencia habría entre ambos enfoques en cuanto a cómo se construye la consulta SQL final.

---

## ✅ Cierre

Tu listado de videojuegos ya filtra por plataforma dentro de un JSON, combinado de forma transparente con el resto de filtros relacionales. En la próxima actividad haces pruebas de integración reales sobre todo lo construido en este tema: persistencia y consultas JSONB juntas, contra un PostgreSQL de verdad.
