<a id="jdbc-puro"></a>

# 🧩 4. JDBC puro: conexión manual sin ORM

Todo lo que has hecho hasta ahora con la base de datos ha pasado por Spring Data JPA: declaras una entidad, extiendes `JpaRepository`, y `save()`/`findAll()` funcionan sin que escribas una sola línea de SQL. Hoy vas a levantar esa capa y ver qué hay debajo — la API sobre la que se apoya **todo** lo demás, incluido Hibernate: **JDBC**.

---

## 🧱 Las cuatro piezas de JDBC

**JDBC** (*Java Database Connectivity*) es la API estándar de Java (`java.sql`) para hablar con bases de datos relacionales — ya la mencionaste en el apartado 2 como el protocolo que implementa cada driver. Se apoya en cuatro piezas, cada una con un papel concreto:

| Pieza | Papel |
|---|---|
| `DriverManager` / `DataSource` | Abre la conexión con el gestor. |
| `Connection` | Representa esa sesión abierta con la base de datos. |
| `Statement` / `PreparedStatement` | Transporta el SQL que quieres ejecutar. |
| `ResultSet` | El cursor sobre las filas que devuelve una consulta. |

Cada una de estas piezas es un **recurso**: mientras está abierta, mantiene ocupada una conexión de red y memoria en el gestor de base de datos. Un recurso que se abre y no se cierra no desaparece solo — sigue consumiendo esos recursos hasta que algo lo cierre explícitamente (o, en el peor caso, hasta que el proceso entero termine).

---

## 🔧 El ciclo completo, paso a paso

### 1. Abrir la conexión

```java
Connection conn = DriverManager.getConnection(
        "jdbc:postgresql://localhost:5432/libreria_db",
        "libreria_user",
        "password123"
);
```

`DriverManager.getConnection(...)` recibe la misma URL JDBC, usuario y contraseña que ya conoces de `application-dev.yaml` — solo que aquí los pasas directamente en Java, sin que Spring medie. En un proyecto real, en vez de `DriverManager` se usaría casi siempre un `DataSource` con pooling (como el HikariCP que Spring configura automáticamente por ti) — `DriverManager` abre una conexión nueva cada vez, sin reutilizar nada.

### 2. Preparar la sentencia — y por qué nunca con `+`

```java
String sql = "SELECT id, titulo, precio FROM libro WHERE editorial_id = ?";
PreparedStatement stmt = conn.prepareStatement(sql);
stmt.setLong(1, editorialId);
```

!!! danger "Nunca concatenes el SQL con datos del usuario"
    Podrías escribir `"SELECT * FROM libro WHERE editorial_id = " + editorialId` y funcionaría... hasta que `editorialId` (o cualquier dato que venga de fuera) contenga algo como `1 OR 1=1`. Eso es una **inyección SQL**: el atacante consigue que su texto se interprete como código SQL, no como un simple valor. `PreparedStatement` con parámetros (`?`) evita esto por diseño: el valor se envía por separado del SQL, y el driver lo trata siempre como un dato, nunca como código a ejecutar, sea lo que sea lo que contenga.

### 3. Ejecutar y recorrer el resultado

```java
ResultSet rs = stmt.executeQuery();
while (rs.next()) {
    Long id = rs.getLong("id");
    String titulo = rs.getString("titulo");
    BigDecimal precio = rs.getBigDecimal("precio");
    // mapear a mano a un objeto Java
}
```

`ResultSet` es un cursor: `rs.next()` avanza fila a fila (y devuelve `false` cuando ya no quedan más), y `rs.getXxx("columna")` extrae cada valor de la fila actual, con el tipo Java que corresponda.

### 4. Cerrar los recursos — en orden, o con try-with-resources

```java
try (Connection conn = DriverManager.getConnection(url, user, pass);
     PreparedStatement stmt = conn.prepareStatement(sql)) {

    stmt.setLong(1, editorialId);
    try (ResultSet rs = stmt.executeQuery()) {
        while (rs.next()) {
            // ...
        }
    }
}
```

`try-with-resources` cierra automáticamente cada recurso al salir del bloque (en orden inverso a como se abrieron), sin que tengas que escribir un `finally` con cada `close()` a mano — y lo hace incluso si se lanza una excepción a mitad. Sin este cierre, cada conexión que "olvidas" cerrar queda ocupada indefinidamente: si esto se repite, acabas agotando el *pool* de conexiones que viste en el apartado 2, y la aplicación deja de poder conectarse a la base de datos.

---

## 🆚 Lo que Spring Data JPA te estaba ahorrando

Compara este ciclo completo con lo que ya conoces de `operaciones-crud-transacciones.md`:

```java
// Con Spring Data JPA — todo esto está oculto:
List<Libro> libros = libroRepository.findAll();
```

Una sola línea, sin gestionar conexión, sin escribir SQL, sin mapear filas a mano, sin preocuparte de cerrar nada — Spring Data JPA hace todo eso por ti, usando JDBC exactamente igual por debajo. Ahora que has visto la "fontanería" real, esa línea deja de ser magia: es JDBC con un montón de trabajo repetitivo automatizado.

!!! tip "¿Entonces por qué aprender JDBC puro, si un ORM lo hace todo?"
    Porque un ORM sigue generando SQL por debajo, y entender qué SQL genera (y por qué a veces conviene evitarlo) te va a ayudar a diagnosticar problemas de rendimiento más adelante en tu carrera. Además, hay situaciones — consultas muy específicas, procedimientos almacenados (que verás en el siguiente apartado) — donde JDBC puro, o herramientas intermedias como `JdbcTemplate`, siguen siendo la opción más directa.

En este punto del curso, tu propio proyecto usa Spring Data JPA en todas partes, sin una sola línea de JDBC manual (tus repositorios son interfaces vacías que extienden `JpaRepository`). Por eso, en la Actividad 1.3, vas a escribir esta consulta manual en una clase de consola aparte, fuera de tus controladores y servicios, conectándote al mismo PostgreSQL que ya tienes levantado.

---

## ✅ Ideas clave

??? tip "Abrir resumen"

    - **JDBC** es la API estándar de Java para bases de datos, y la base sobre la que se apoya cualquier ORM, incluido Hibernate.
    - Sus cuatro piezas: `DriverManager`/`DataSource` (abre conexión), `Connection` (la sesión), `Statement`/`PreparedStatement` (transporta el SQL), `ResultSet` (cursor sobre el resultado).
    - Un **recurso** abierto y no cerrado sigue consumiendo conexión y memoria — por eso se cierra siempre, idealmente con `try-with-resources`.
    - `PreparedStatement` con parámetros (`?`) evita la **inyección SQL**; concatenar SQL con datos externos (`+`) es peligroso siempre.
    - Todo lo que hace `@Transactional`/`JpaRepository` de forma automática (abrir conexión, ejecutar, cerrar, mapear filas a objetos) es exactamente esto, hecho a mano.
