# 🧪 Actividad 2.1: La columna `detallesPlataforma` — persistiendo objetos estructurados

!!! info "Práctica guiada"
    Añades a tu `Videojuego` una columna JSONB real, la expones en tu API, y comprueba con tus propios ojos que PostgreSQL la guarda como JSON de verdad, no como texto plano.

## Qué vas a practicar

- Añadir un campo JSONB a una entidad ya existente.
- Ampliar DTOs para exponer ese campo en la API.
- Verificar en la base de datos que el tipo de columna es realmente `jsonb`.
- Entender el comportamiento de reemplazo total al actualizar.

---

## Requisitos previos

Tu entidad `Videojuego` y su CRUD completo (Tema 1).

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

Y en el `mapToDTO`/`create`/`update` de tu `VideojuegoService`, asegúrate de que este campo se propaga igual que los demás (`v.setDetallesPlataforma(dto.detallesPlataforma())`, y en el DTO de respuesta).

Borra la tabla `videojuego` (o usa una base de datos de prueba) para que Hibernate la recree con la columna nueva, y reinicia tu aplicación.

!!! warning "Esto rompe el test MockMvc que ya tienes de PSP"
    `VideojuegoControllerTest` (Programación de Servicios y Procesos, Actividad 1.3) construye un `VideojuegoResponseDTO` con el constructor de los campos antiguos — al añadir `detallesPlataforma` como sexto campo, ese `new VideojuegoResponseDTO(...)` deja de compilar. Actualiza esa llamada en el test, añadiendo un último argumento (por ejemplo, `Map.of()` para un videojuego sin plataformas).

---

## Paso 2 — Crear un videojuego con una plataforma

```bash
curl -X POST http://localhost:8080/api/v1/videojuegos \
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

### Mini-reto — varias plataformas anidadas

Sin más indicaciones, crea un segundo videojuego, esta vez con `detallesPlataforma` conteniendo **al menos dos** plataformas distintas, con las claves que tú elijas dentro de cada una (id de la app, número de logros, o lo que consideres relevante). Verifica igual que en el paso anterior.

---

## Paso 3 — Verificación en la base de datos

```bash
docker exec -it <tu-contenedor-postgres> psql -U gamevault_user -d gamevault_db -c "\d videojuego"
```

**Comprueba** que la columna `detallesplataforma` (Postgres normaliza a minúsculas) aparece con tipo `jsonb`, no `text` ni `character varying`.

Consulta el contenido directamente en SQL:

```bash
docker exec -it <tu-contenedor-postgres> psql -U gamevault_user -d gamevault_db \
  -c "SELECT titulo, detallesplataforma FROM videojuego;"
```

**Anota**: ¿el JSON que ves en la consola de PostgreSQL coincide exactamente con el que mandaste por la API?

---

## Paso 4 — Actualizar y observar el reemplazo total

Actualiza el primer videojuego con un `PUT`, cambiando `detallesPlataforma` a una estructura **distinta** (por ejemplo, solo con la clave `"switch"`, sin `"steam"`):

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

**Comprueba**: consultando de nuevo en `psql`, ¿sigue estando `"steam"` en el JSON guardado, o ha desaparecido por completo?

**Pregunta de comprensión**: ¿por qué el `PUT` reemplaza el objeto JSON completo en vez de combinar (*merge*) el nuevo contenido con el anterior? Relaciona tu respuesta con cómo funciona el mapeo `Map` → `jsonb`: ¿en qué punto del proceso Hibernate tendría que decidir "combinar" en vez de "sustituir", y por qué no lo hace por defecto?

---

## Pregunta final

¿Qué ventaja concreta tiene JSONB frente a crear una tabla `plataforma_videojuego` (con una fila por plataforma y sus columnas propias)? ¿En qué situación sería mejor la tabla relacional en vez de JSONB — piensa en un caso donde necesitaras, por ejemplo, buscar todos los videojuegos disponibles en una plataforma concreta de forma muy eficiente, o donde la estructura de cada plataforma tuviera que cumplir reglas estrictas?

---

## ✅ Cierre

Tu `Videojuego` ya persiste objetos estructurados reales en una columna JSONB. Todavía no has consultado por su contenido — solo lo has guardado y leído entero. En la próxima actividad vas a filtrar videojuegos según qué plataformas tienen, usando `jsonb_exists`.
