# 🧪 Actividad 3.2: Filtrar por plataforma con `jsonb_exists`

!!! info "Práctica guiada"
    Añades el filtro por plataforma a tu listado de videojuegos, usando `jsonb_exists` sobre la columna JSONB de la Actividad 3.1.

## Qué vas a practicar

- Invocar una función SQL nativa desde Criteria API con `criteriaBuilder.function(...)`.
- Combinar un filtro JSONB con los filtros ya existentes.
- Verificar el comportamiento de modificación de un campo JSONB filtrado.

---

## Requisitos previos

Tu columna `detallesPlataforma` de la Actividad 3.1, con videojuegos de distintas plataformas ya creados; tus Specifications de la Actividad 2.2.

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

Por qué cada pieza: `criteriaBuilder.function("jsonb_exists", Boolean.class, ...)` invoca esa función nativa de PostgreSQL, indicando que devuelve un `Boolean`; `root.get("detallesPlataforma")` es la columna sobre la que se aplica; `criteriaBuilder.literal(plataforma.toLowerCase())` es la clave a buscar, pasada como literal (no como columna) — y en minúsculas, para que la búsqueda no dependa de cómo escribió el usuario la plataforma (asumiendo que también guardas las claves del JSON en minúsculas, como en tus datos de la Actividad 3.1). `criteriaBuilder.isTrue(...)` envuelve el resultado booleano de la función para usarlo como condición del `WHERE`.

---

## Paso 2 — Añadir el filtro al DTO y combinarlo — mini-reto

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

Ahora, sin más indicaciones, encadena `disponibleEnPlataforma(filtro.plataforma())` con `.and(...)` en `findAllPaginated()` de tu `VideojuegoService` — es exactamente el mismo patrón que ya repetiste varias veces en la Actividad 2.2, así que no necesitas código nuevo mostrado: solo añadir una línea más a la cadena que ya tienes.

---

## Paso 3 — Prueba con peticiones reales

```bash
# Plataforma que SÍ existe en tus datos de prueba
curl "http://localhost:8080/api/v1/videojuegos?plataforma=steam"

# Plataforma que NO existe en ningún videojuego
curl "http://localhost:8080/api/v1/videojuegos?plataforma=xbox"
```

**Comprueba**: que el primer caso devuelve solo los videojuegos que de verdad tienen la clave `"steam"` en su `detallesPlataforma`, y el segundo devuelve una lista vacía (no un error).

---

## Paso 4 — Modificar y comprobar que el filtro reacciona

Actualiza uno de tus videojuegos que sí aparecía en el filtro por `steam`, quitando esa clave de su `detallesPlataforma`:

```bash
curl -X PUT http://localhost:8080/api/v1/videojuegos/1 \
  -H "Content-Type: application/json" \
  -d '{
    "titulo": "Celeste",
    "precio": 19.99,
    "fechaLanzamiento": "2018-01-25",
    "estudioId": 1,
    "detallesPlataforma": {"switch": {"idApp": "ABCD"}}
  }'
```

Repite el filtro por `steam`:

```bash
curl "http://localhost:8080/api/v1/videojuegos?plataforma=steam"
```

**Comprueba**: que ese videojuego ya no aparece en el resultado, porque su `detallesPlataforma` ya no contiene la clave `"steam"` (recuerda: el `PUT` reemplaza el objeto completo, como viste en la Actividad 3.1).

---

## Pregunta final

¿Qué haría falta para filtrar por "disponible en Steam **Y** en Switch a la vez"? Piensa en dos enfoques posibles: encadenar dos veces `disponibleEnPlataforma` con `.and(...)` (una vez por cada plataforma) frente a usar una función distinta de PostgreSQL como `jsonb_exists_all` (que comprueba varias claves de golpe). No hace falta que implementes ninguno de los dos — explica con tus palabras qué diferencia habría entre ambos enfoques en cuanto a cómo se construye la consulta SQL final.

---

## ✅ Cierre

Tu listado de videojuegos ya filtra por plataforma dentro de un JSON, combinado de forma transparente con el resto de filtros relacionales. En la próxima actividad haces pruebas de integración reales sobre todo lo construido en este tema: persistencia y consultas JSONB juntas, contra un PostgreSQL de verdad.
