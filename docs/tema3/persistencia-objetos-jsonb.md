<a id="persistencia-objetos-jsonb"></a>

# 🧩 1. Persistencia de objetos con JSONB

Hasta ahora, cada campo de tus entidades ha sido un valor simple: un texto, un número, una fecha. Este tema abre una categoría intermedia entre lo puramente relacional (Temas 1-2) y lo orientado a objetos puro (que verás de forma teórica más adelante en este mismo tema): las **bases de datos objeto-relacionales**, gestores relacionales que además saben almacenar y consultar objetos completos dentro de sus tablas.

---

## 📦 Repaso rápido: qué es JSON

Ya lo has visto antes, pero conviene no darlo por dominado. **JSON** (*JavaScript Object Notation*) es un formato de texto para representar datos estructurados: objetos (pares clave-valor entre `{}`), arrays (listas entre `[]`), y valores anidables entre ambos:

```json
{
  "steam": { "idApp": 123, "logros": 40 },
  "switch": { "idApp": "ABCD" }
}
```

Se ha convertido en el formato universal de intercambio de datos porque es legible por humanos, ligero, y prácticamente todos los lenguajes de programación saben leerlo y escribirlo de forma nativa o casi nativa.

---

## 🧱 Objeto simple vs. objeto estructurado

- **Objeto simple**: un valor escalar — un texto, un número, una fecha. Es lo que has persistido siempre (`titulo`, `precio`).
- **Objeto estructurado**: un objeto con campos anidados, posiblemente con colecciones dentro — como el ejemplo JSON de arriba, con dos plataformas, cada una con sus propios campos internos.

---

## 🐘 Qué ofrece PostgreSQL: el tipo `jsonb`

PostgreSQL tiene un tipo de columna especial, **`jsonb`** (JSON binario), que permite que una fila normal, de una tabla normal, lleve dentro un objeto JSON completo — con estructura que puede variar de una fila a otra — conviviendo con las columnas y claves foráneas de siempre. No hace falta una base de datos documental aparte (eso lo verás en el Tema 4) para tener esta flexibilidad en casos puntuales.

!!! tip "`json` vs. `jsonb`, en una frase"
    PostgreSQL tiene dos tipos: `json` guarda el texto tal cual, sin analizarlo; `jsonb` lo analiza y lo guarda en un formato binario indexable y más rápido de consultar — por eso `jsonb` es casi siempre la opción preferida, salvo que necesites preservar el texto original exacto (espacios, orden de claves).

---

## 🎮 Aterrizaje en GameVault: `detallesPlataforma`

### El campo, con sus anotaciones

```java
@Entity
@Table(name = "videojuego")
public class Videojuego {

    // ... id, titulo, precio, fechaLanzamiento (objetos simples, ya conocidos) ...

    @JdbcTypeCode(SqlTypes.JSON)
    @Column(columnDefinition = "jsonb")
    private Map<String, Object> detallesPlataforma;
}
```

`@Column(columnDefinition = "jsonb")` le dice a Hibernate qué tipo de columna PostgreSQL usar; `@JdbcTypeCode(SqlTypes.JSON)` le dice **cómo** tratar ese campo — como JSON, no como un texto plano cualquiera. El tipo Java es un simple `Map<String, Object>`: no necesitas una clase Java específica para cada estructura posible, el mapa acepta cualquier forma de JSON.

### `titulo`/`precio` (objeto simple) vs. `detallesPlataforma` (objeto estructurado)

`titulo` y `precio` son la persistencia "de toda la vida" que ya conoces del Tema 2 — un valor escalar por columna. `detallesPlataforma`, en cambio, es un objeto estructurado real: puede contener claves anidadas como

```json
{"steam": {"idApp": 123, "logros": 40}, "switch": {"idApp": "ABCD"}}
```

todo dentro de una única columna, sin que hayas tenido que crear una tabla nueva por cada plataforma que el juego soporte.

### Cómo Hibernate serializa el `Map` a JSON

`com/aleroig/gamevault/config/Jackson3HibernateMapper.java` es la pieza de configuración que permite que Hibernate sepa convertir ese `Map<String, Object>` Java a JSON (y de vuelta) usando Jackson, la librería estándar de JSON en el ecosistema Spring — no necesitas entender el detalle exacto de esa clase, solo saber que existe porque Hibernate necesita esa ayuda extra para mapear un tipo que no es directo como un `String` o un `BigDecimal`.

---

## ⚖️ Por qué JSONB y no una tabla nueva

Con lo que ya sabes del Tema 2, podrías haber modelado esto como una tabla `detalle_plataforma` con una relación `@OneToMany` desde `Videojuego` — una fila por plataforma, con sus propias columnas. ¿Por qué el proyecto eligió JSONB en su lugar?

| | Tabla relacional nueva | Columna JSONB |
|---|---|---|
| **Añadir una plataforma nueva** | Requiere migración de esquema (nueva tabla o columnas) | No requiere ningún cambio de esquema |
| **Integridad referencial** | Garantizada por el motor (claves foráneas) | Ninguna — el contenido del JSON no se valida contra nada |
| **Tipado estricto** | Cada columna tiene su tipo fijo | Sin tipado — cualquier estructura es válida |

El trade-off es real: JSONB gana en flexibilidad (añadir Xbox o una plataforma nueva mañana no toca el esquema de la base de datos) a cambio de perder las garantías que sí tendría una tabla normal. Es una decisión de diseño, no una regla universal — y es justo lo que vas a discutir con criterio en la Actividad 3.1.

---

## 🔗 Sobre "establecer y cerrar conexiones" con JSONB

Conviene ser explícito sobre el establecimiento y cierre de conexiones: JSONB **no abre ninguna conexión nueva**. Es el mismo PostgreSQL, el mismo `DataSource` configurado desde el Tema 1, gestionado exactamente igual por el framework. No hay nada especial que conectar o cerrar para trabajar con una columna JSONB — es una columna más de una tabla que ya conocías.

---

## ✅ Ideas clave

??? tip "Abrir resumen"

    - Una base de datos **objeto-relacional** es un gestor relacional que además sabe almacenar objetos completos dentro de sus tablas.
    - `jsonb` en PostgreSQL guarda JSON en formato binario indexable — preferible a `json` (texto plano) salvo que necesites preservar el formato exacto.
    - `@JdbcTypeCode(SqlTypes.JSON)` + `@Column(columnDefinition = "jsonb")` mapean un `Map<String, Object>` Java contra una columna `jsonb`.
    - **Objeto simple** = valor escalar; **objeto estructurado** = objeto anidado con posibles colecciones dentro.
    - JSONB gana en flexibilidad de esquema, pierde integridad referencial y tipado estricto frente a una tabla relacional — es una decisión de diseño con trade-offs, no una opción siempre superior.
    - JSONB no abre ninguna conexión nueva: reutiliza el mismo `DataSource` del Tema 1.
