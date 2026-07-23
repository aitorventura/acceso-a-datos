<a id="persistencia-objetos-jsonb"></a>

# 🧩 1. Persistencia de objetos con JSONB

Hasta ahora, cada campo de tus entidades ha sido un valor simple: un texto, un número, una fecha. Este tema abre una categoría intermedia entre lo puramente relacional (Tema 1) y lo orientado a objetos puro (que verás de forma teórica más adelante en este mismo tema): las **bases de datos objeto-relacionales**, gestores relacionales que además saben almacenar y consultar objetos completos dentro de sus tablas.

---

## 📦 Repaso rápido: qué es JSON

Ya lo has visto antes, pero conviene no darlo por dominado. **JSON** (*JavaScript Object Notation*) es un formato de texto para representar datos estructurados: objetos (pares clave-valor entre `{}`), arrays (listas entre `[]`), y valores anidables entre ambos:

```json
{
  "tapaDura": { "isbn": "978-84-376-0494-7", "paginas": 496 },
  "ebook": { "formato": "epub" }
}
```

Se ha convertido en el formato universal de intercambio de datos porque es legible por humanos, ligero, y prácticamente todos los lenguajes de programación saben leerlo y escribirlo de forma nativa o casi nativa.

---

## 🧱 Objeto simple vs. objeto estructurado

- **Objeto simple**: un valor escalar — un texto, un número, una fecha. Es lo que has persistido siempre (`titulo`, `precio`).
- **Objeto estructurado**: un objeto con campos anidados, posiblemente con colecciones dentro — como el ejemplo JSON de arriba, con dos formatos de edición, cada uno con sus propios campos internos.

---

## 🐘 Qué ofrece PostgreSQL: el tipo `jsonb`

PostgreSQL tiene un tipo de columna especial, **`jsonb`** (JSON binario), que permite que una fila normal, de una tabla normal, lleve dentro un objeto JSON completo — con estructura que puede variar de una fila a otra — conviviendo con las columnas y claves foráneas de siempre. No hace falta una base de datos documental aparte (eso lo verás en el Tema 3) para tener esta flexibilidad en casos puntuales, ni tampoco una conexión distinta: sigue siendo el mismo PostgreSQL, el mismo `DataSource` que configuraste en el Tema 1, gestionado exactamente igual — una columna `jsonb` no abre ni cierra nada por su cuenta.

!!! tip "`json` vs. `jsonb`, en una frase"
    PostgreSQL tiene dos tipos: `json` guarda el texto tal cual, sin analizarlo; `jsonb` lo analiza y lo guarda en un formato binario indexable y más rápido de consultar — por eso `jsonb` es casi siempre la opción preferida, salvo que necesites preservar el texto original exacto (espacios, orden de claves).

---

## 📚 Un ejemplo completo: `detallesEdicion`

Siguiendo con la librería: cada libro puede existir en varios formatos (tapa dura, bolsillo, ebook), cada uno con sus propios datos internos. Ese es un objeto estructurado dentro de la entidad `Libro`.

### El campo, con sus anotaciones

```java
@Entity
@Table(name = "libro")
public class Libro {

    // ... id, titulo, precio, fechaPublicacion (objetos simples, ya conocidos) ...

    @JdbcTypeCode(SqlTypes.JSON)
    @Column(columnDefinition = "jsonb")
    private Map<String, Object> detallesEdicion;
}
```

`@Column(columnDefinition = "jsonb")` le dice a Hibernate qué tipo de columna PostgreSQL usar; `@JdbcTypeCode(SqlTypes.JSON)` le dice **cómo** tratar ese campo — como JSON, no como un texto plano cualquiera. El tipo Java es un simple `Map<String, Object>`: no necesitas una clase Java específica para cada estructura posible, el mapa acepta cualquier forma de JSON.

### `titulo`/`precio` (objeto simple) vs. `detallesEdicion` (objeto estructurado)

`titulo` y `precio` son la persistencia "de toda la vida" que ya conoces del Tema 1 — un valor escalar por columna. `detallesEdicion`, en cambio, es un objeto estructurado real: puede contener claves anidadas como

```json
{"tapaDura": {"isbn": "978-84-376-0494-7", "paginas": 496}, "ebook": {"formato": "epub"}}
```

todo dentro de una única columna, sin que hayas tenido que crear una tabla nueva por cada formato en que se publique el libro.

### Cómo Hibernate serializa el `Map` a JSON

Hibernate no convierte un `Map<String, Object>` Java a JSON (y de vuelta) por sí solo — necesita apoyarse en **Jackson**, la librería estándar de JSON en el ecosistema Spring. La buena noticia es que no tienes que configurar nada de eso a mano: Spring Boot incluye Jackson en el classpath por defecto, y en cuanto Hibernate 6 detecta un campo anotado con `@JdbcTypeCode(SqlTypes.JSON)`, lo conecta automáticamente — ni una clase de configuración que escribir, ni ninguna dependencia nueva que añadir.

---

## ⚖️ Por qué JSONB y no una tabla nueva

Con lo que ya sabes del Tema 1, podrías haber modelado esto como una tabla `detalle_edicion` con una relación `@OneToMany` desde `Libro` — una fila por formato, con sus propias columnas. ¿Por qué elegir JSONB en su lugar?

| | Tabla relacional nueva | Columna JSONB |
|---|---|---|
| **Añadir un formato nuevo** | Requiere migración de esquema (nueva tabla o columnas) | No requiere ningún cambio de esquema |
| **Integridad referencial** | Garantizada por el motor (claves foráneas) | Ninguna — el contenido del JSON no se valida contra nada |
| **Tipado estricto** | Cada columna tiene su tipo fijo | Sin tipado — cualquier estructura es válida |

El trade-off es real: JSONB gana en flexibilidad (añadir un formato nuevo mañana — audiolibro, por ejemplo — no toca el esquema de la base de datos) a cambio de perder las garantías que sí tendría una tabla normal. Es una decisión de diseño, no una regla universal — y es justo lo que vas a discutir con criterio en la Actividad 2.1.

!!! example "La pérdida de tipado, en la práctica"
    Con una tabla relacional, escribir `detalleEdicion.getPaginas()` que no existe da un error de compilación — no hay forma de que el código llegue a ejecutarse así. Con `Map<String, Object>`, en cambio, `detallesEdicion.get("paginas")` compila sin problema aunque hayas escrito `"pagina"` por error: el fallo no aparece hasta que ejecutas el código y te encuentras con un `null` inesperado (o un `ClassCastException`, si además asumías un tipo concreto). El compilador ya no te cubre las espaldas — pasa a ser responsabilidad tuya, en tiempo de ejecución.

---

## 📇 Indexar el contenido de una columna JSONB

Un **índice** es una estructura auxiliar que el motor de base de datos mantiene aparte de la tabla, pensada para encontrar filas rápido sin tener que recorrerlas todas una a una — la misma idea que el índice de un libro: en vez de leer página por página buscando un tema, vas directo a la página que el índice te señala. Sin ningún índice, cualquier búsqueda por una columna implica recorrer la tabla entera, fila a fila.

El tipo de índice más habitual es el **B-tree**, y acelera búsquedas por el valor completo de una columna — perfecto para `WHERE titulo = 'Celeste'`. Pero JSONB no tiene "un valor": tiene un objeto con claves dentro, y lo que sueles necesitar filtrar es "¿existe esta clave?" o "¿contiene este valor concreto?" — algo que un índice B-tree normal no sabe acelerar, porque no entiende la estructura interna del JSON.

PostgreSQL resuelve esto con un tipo de índice distinto: **GIN** (*Generalized Inverted Index*), pensado precisamente para tipos de datos "compuestos" como JSONB, arrays, o búsqueda de texto completo. Se crea así:

```sql
CREATE INDEX idx_libro_detalles_edicion ON libro USING GIN (detalles_edicion);
```

Con este índice, una consulta que filtre por el contenido del JSON (como `jsonb_exists`, que verás en el próximo apartado) deja de recorrer la tabla entera fila a fila — PostgreSQL consulta el índice directamente. Pero eso solo pasa si de verdad compensa: hacen falta **dos** condiciones a la vez, no solo una. Necesitas volumen (con pocas filas, cualquier plan es rápido) **y** que la consulta sea selectiva — que descarte la mayoría de la tabla, no la mitad. Si una consulta devuelve, por ejemplo, el 50% de las filas, recorrerlas todas seguidas sigue siendo más barato que saltar de una en una por el índice, por muchas filas que tenga la tabla. Es exactamente el tipo de decisión que separa una consulta que escala de una que no.

!!! tip "Con pocas filas no se nota — por eso en la actividad generas muchas"
    Con una tabla de prueba de un puñado de filas no verías ninguna diferencia real: el motor decide el plan por coste estimado, y con pocos datos un recorrido completo de la tabla ya es barato de por sí. En la Actividad 2.1 vas a generar 100.000 filas de golpe y comparar el plan de la misma consulta antes y después de crear el índice — con ese volumen, la diferencia deja de ser cuestión de suerte.

---

## ⚠️ `NULL` de la columna vs. `null` dentro del JSON

Con JSONB hay tres estados distintos que se confunden con facilidad:

| Caso | Qué significa |
|---|---|
| La columna `detalles_edicion` es `NULL` | El libro no tiene ningún dato de edición guardado — ni siquiera un objeto vacío. |
| `{"ebook": null}` | Hay un objeto JSON guardado, con la clave `"ebook"` presente, pero su valor es el `null` de JSON — no confundir con el `NULL` de SQL de arriba. |
| `{}` (objeto vacío) | Hay un objeto JSON guardado, sin ninguna clave dentro. |

Comprobar "¿existe la clave `ebook`?" da resultados distintos en cada caso: en el primero, la consulta ni siquiera tiene sobre qué mirar; en el segundo, la clave existe, aunque su valor sea `null`; en el tercero, la clave directamente no existe. Tenerlo claro desde ahora te ahorra un bug real: "el libro no tiene ebook" no es lo mismo que "la clave `ebook` no existe" — podrías tener la clave con valor `null` a propósito, para indicar "hay edición ebook prevista, pero sin detalles todavía".

---

## ✅ Ideas clave

??? tip "Abrir resumen"

    - Una base de datos **objeto-relacional** es un gestor relacional que además sabe almacenar objetos completos dentro de sus tablas.
    - `jsonb` en PostgreSQL guarda JSON en formato binario indexable — preferible a `json` (texto plano) salvo que necesites preservar el formato exacto.
    - `@JdbcTypeCode(SqlTypes.JSON)` + `@Column(columnDefinition = "jsonb")` mapean un `Map<String, Object>` Java contra una columna `jsonb`.
    - **Objeto simple** = valor escalar; **objeto estructurado** = objeto anidado con posibles colecciones dentro.
    - JSONB gana en flexibilidad de esquema, pierde integridad referencial y tipado estricto frente a una tabla relacional — es una decisión de diseño con trade-offs, no una opción siempre superior.
    - JSONB no abre ninguna conexión nueva: reutiliza el mismo `DataSource` del Tema 1.
    - Un índice **GIN** acelera consultas por el contenido de una columna JSONB — un índice normal (B-tree) no sabe mirar dentro del objeto. Pero solo gana si la consulta es selectiva (descarta la mayoría de filas) además de haber volumen — con una consulta poco selectiva, el `Seq Scan` sigue siendo más barato por muchas filas que haya.
    - `NULL` de columna, `null` dentro del JSON, y objeto vacío `{}` son tres estados distintos, y comprobar "existe la clave" da un resultado diferente en cada uno.
