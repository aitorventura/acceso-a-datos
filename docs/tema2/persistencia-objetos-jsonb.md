<a id="persistencia-objetos-jsonb"></a>

# 🧩 1. Persistencia de objetos con JSONB

![Persistencia de objetos con JSONB](diapositivas/persistencia-objetos-jsonb.pdf){ type=application/pdf style="width:100%;min-height:80vh" }

!!!info "Descarga de diapositivas"
    [Descarga las diapositivas](diapositivas/persistencia-objetos-jsonb.pptx){target="_blank" rel="noopener"}

---

Hasta ahora, cada atributo de tus entidades se almacenaba normalmente en una columna independiente: un texto, un número o una fecha. Este apartado introduce una posibilidad intermedia entre el modelo relacional clásico y las bases de datos orientadas a objetos: utilizar capacidades objeto-relacionales de PostgreSQL para almacenar datos estructurados dentro de una columna.

Con `jsonb`, PostgreSQL no guarda directamente un objeto Java ni sus métodos. Guarda su representación como un documento JSON, que Hibernate puede convertir al escribir y reconstruir al leer.

---

## 📦 Repaso rápido: qué es JSON

Ya lo has visto antes, pero conviene no darlo por dominado. **JSON** (*JavaScript Object Notation*) es un formato de texto para representar datos estructurados: objetos (pares clave-valor entre `{}`), arrays (listas entre `[]`), y valores anidables entre ambos:

```json
{
   "tapaDura":{
      "isbn":"978-84-376-0494-7",
      "paginas":496
   },
   "ebook":{
      "formato":"epub"
   }
}
```

Se ha convertido en el formato universal de intercambio de datos porque es legible por humanos, ligero, y prácticamente todos los lenguajes de programación saben leerlo y escribirlo de forma nativa o casi nativa.

---

## 🧱 Objeto simple vs. objeto estructurado

- **Objeto simple**: un valor escalar — un texto, un número, una fecha. Es lo que has persistido siempre (`titulo`, `precio`).
- **Objeto estructurado**: un objeto con campos anidados, posiblemente con colecciones dentro. En el ejemplo anterior, se almacenan los datos de dos ediciones del mismo libro —`tapaDura` y `ebook`— y cada una tiene sus propios atributos.

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

`titulo` y `precio` son la persistencia "de toda la vida" que ya conoces del Tema 1 — un valor escalar por columna. `detallesEdicion`, en cambio, representa datos estructurados que Hibernate serializa como JSON dentro de una única columna:

```json
{
   "tapaDura":{
      "isbn":"978-84-376-0494-7",
      "paginas":496
   },
   "ebook":{
      "formato":"epub"
   }
}
```

### Cómo Hibernate serializa el `Map` a JSON

Hibernate necesita una biblioteca capaz de convertir el `Map<String, Object>` a JSON y reconstruirlo después. En este proyecto puede utilizar **Jackson**, que ya está disponible porque Spring Boot lo incorpora mediante las dependencias web que añadiste anteriormente.

Al detectar `@JdbcTypeCode(SqlTypes.JSON)` y encontrar Jackson en el classpath, Hibernate lo utiliza para realizar esa conversión. Por eso aquí no necesitas añadir una dependencia ni escribir un conversor manual.

---

## ⚖️ Por qué JSONB y no una tabla nueva

Con lo que ya sabes del Tema 1, podrías haber modelado esto como una tabla `detalle_edicion` con una relación `@OneToMany` desde `Libro` — una fila por formato, con sus propias columnas. ¿Por qué elegir JSONB en su lugar?

| | Modelo relacional | Columna JSONB |
|---|---|---|
| **Añadir otra edición con la misma estructura** | Basta con insertar otra fila | Basta con modificar el documento JSON |
| **Añadir atributos o estructuras nuevas** | Puede requerir nuevas columnas, tablas o relaciones | Normalmente no requiere modificar el esquema |
| **Integridad y validación** | El motor aplica tipos, restricciones y claves foráneas | La estructura interna no se valida por defecto |
| **Acceso desde Java** | Entidades y atributos tipados | Con `Map<String, Object>`, claves y tipos se comprueban durante la ejecución |

El equilibrio es real: JSONB facilita que los datos internos evolucionen sin modificar continuamente el esquema de tablas, pero reduce el tipado y las garantías que PostgreSQL aplica automáticamente sobre cada campo. Un modelo relacional aporta más control; JSONB, más flexibilidad. La elección es una decisión de diseño, no existe una regla universal — y es justo lo que vas a discutir con criterio en la Actividad 2.1.

!!! example "La pérdida de tipado, en la práctica"
    Con una tabla relacional, escribir `detalleEdicion.getPaginas()` que no existe da un error de compilación — no hay forma de que el código llegue a ejecutarse así. Con `Map<String, Object>`, en cambio, `detallesEdicion.get("paginas")` compila sin problema aunque hayas escrito `"pagina"` por error: el fallo no aparece hasta que ejecutas el código y te encuentras con un `null` inesperado (o un `ClassCastException`, si además asumías un tipo concreto). El compilador ya no te cubre las espaldas — pasa a ser responsabilidad tuya, en tiempo de ejecución.

---

## 📇 Indexar el contenido de una columna JSONB

Un **índice** es una estructura auxiliar que el motor de base de datos mantiene aparte de la tabla, pensada para encontrar filas rápido sin tener que recorrerlas todas una a una — la misma idea que el índice de un libro: en vez de leer página por página buscando un tema, vas directo a la página que el índice te señala. Sin un índice adecuado para la consulta, PostgreSQL puede tener que recorrer la tabla entera, fila a fila.

El tipo de índice más habitual es el **B-tree**, apropiado para operaciones como igualdad, comparaciones por rango y ordenación sobre valores escalares: por ejemplo, buscar un título concreto u ordenar por precio.

Pero las consultas habituales sobre JSONB no comparan necesariamente el documento completo. Suelen preguntar si existe una clave, si el documento contiene cierta estructura o si un valor aparece dentro de él.

PostgreSQL resuelve estas búsquedas mediante índices **GIN** (*Generalized Inverted Index*), diseñados para localizar elementos dentro de valores compuestos como JSONB, arrays o documentos de texto. Se crea así:

```sql
CREATE INDEX idx_libro_detalles_edicion
ON libro USING GIN (detalles_edicion);
```
Por ejemplo, el operador `?` permite comprobar si existe una clave en el nivel superior del documento:

```sql
SELECT *
FROM libro
WHERE detalles_edicion ? 'ebook';
```
El índice GIN puede acelerar este tipo de consultas sobre el contenido del JSONB. Eso no significa que PostgreSQL vaya a utilizarlo siempre: el planificador compara los costes y puede preferir recorrer la tabla completa cuando hay pocas filas o cuando la consulta devuelve una parte muy grande de ellas.

Para que el índice resulte especialmente útil suelen coincidir dos condiciones: que exista un volumen considerable de datos y que la consulta sea selectiva, es decir, que descarte la mayoría de las filas.

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

Comprobar «¿existe la clave `ebook`?» da resultados distintos en cada caso: si la columna es `NULL`, la expresión devuelve también `NULL`; en `{"ebook": null}`, la clave existe aunque su valor sea el `null` de JSON; y en `{}`, la clave no existe.

---

## ✅ Ideas clave

??? tip "Abrir resumen"

    - PostgreSQL es un gestor objeto-relacional: mantiene el modelo de tablas y relaciones, pero incorpora tipos y funciones avanzadas, como `JSONB`, para almacenar y consultar datos estructurados.
    - `JSONB` en PostgreSQL guarda JSON en formato binario indexable — preferible a `JSON` (texto plano) salvo que necesites preservar el formato exacto.
    - `@JdbcTypeCode(SqlTypes.JSON)` + `@Column(columnDefinition = "jsonb")` mapean un `Map<String, Object>` Java contra una columna `jsonb`.
    - **Objeto simple** = valor escalar; **objeto estructurado** = objeto anidado con posibles colecciones dentro.
    - `JSONB` gana en flexibilidad de esquema, pierde integridad referencial y tipado estricto frente a una tabla relacional — es una decisión de diseño con trade-offs, no una opción siempre superior.
    - `JSONB` no abre ninguna conexión nueva: reutiliza el mismo `DataSource` del Tema 1.
    - Un índice **GIN** acelera consultas por el contenido de una columna `JSONB` — un índice normal (B-tree) no sabe mirar dentro del objeto. Pero solo gana si la consulta es selectiva (descarta la mayoría de filas) además de haber volumen — con una consulta poco selectiva, el `Seq Scan` sigue siendo más barato por muchas filas que haya.
    - `NULL` de columna, `null` dentro del JSON, y objeto vacío `{}` son tres estados distintos, y comprobar "existe la clave" da un resultado diferente en cada uno.
