# 🧪 Actividad 1.3: Acceso a la base de datos con JDBC puro

!!! info "Práctica guiada"
    Hoy dejas de lado Spring Data JPA por un momento: vas a escribir una clase de consola, aparte de tu GameVault normal, que se conecta a tu PostgreSQL a mano con JDBC puro. Al final vas a comparar ambos enfoques con datos reales delante, no de memoria.

## Qué vas a practicar

- Conectar con `DriverManager` y ejecutar una consulta parametrizada con `PreparedStatement`.
- Recorrer un `ResultSet` mapeando manualmente cada fila.
- Cerrar recursos correctamente con `try-with-resources`.
- Provocar y observar una inyección SQL real, para entender por qué se evita.

---

## Requisitos previos

Tu PostgreSQL de la Actividad 1.1 levantado, con al menos un `Estudio` y varios `Videojuego` asociados (créalos con tu propio CRUD de la Actividad 1.2 si todavía no tienes datos).

---

## Paso 1 — La clase de consola, guiada al completo

Crea una clase nueva, `DiagnosticoJdbc`, en un paquete separado de tu `catalogo` (por ejemplo, `com.tunombre.gamevault.diagnostico`) — no forma parte del flujo normal de Spring, así que no lleva ninguna anotación:

```java
package com.tunombre.gamevault.diagnostico;

import java.sql.*;

public class DiagnosticoJdbc {

    private static final String URL = "jdbc:postgresql://localhost:5432/gamevault_db";
    private static final String USER = "gamevault_user";
    private static final String PASSWORD = "password123";

    public static void main(String[] args) {
        String sql = "SELECT id, titulo, precio FROM videojuego WHERE estudio_id = ?";

        try (Connection conn = DriverManager.getConnection(URL, USER, PASSWORD);
             PreparedStatement stmt = conn.prepareStatement(sql)) {

            stmt.setLong(1, 1L); // cambia el 1L por un id real de tu base de datos

            try (ResultSet rs = stmt.executeQuery()) {
                while (rs.next()) {
                    Long id = rs.getLong("id");
                    String titulo = rs.getString("titulo");
                    var precio = rs.getBigDecimal("precio");
                    System.out.printf("#%d - %s (%.2f€)%n", id, titulo, precio);
                }
            }

        } catch (SQLException e) {
            throw new RuntimeException("Error accediendo a la base de datos", e);
        }
    }
}
```

Ejecuta el `main()` (desde tu IDE, botón derecho → *Run*) con tu aplicación Spring **parada** o en otra ventana — esta clase no necesita que Spring Boot esté arrancado, se conecta directamente a PostgreSQL.

**Comprueba**: que la consola imprime los videojuegos del estudio que hayas puesto, con el formato `#id - título (precio€)`.

---

## Paso 2 — Por qué el cierre importa

Ya está usando `try-with-resources` en el código de arriba. Fíjate en el orden: `Connection` y `PreparedStatement` se abren en el mismo `try`, y `ResultSet` en uno anidado — se cierran automáticamente en orden inverso al salir de cada bloque, incluso si algo falla a mitad.

**Pregunta**: si quitaras el `try-with-resources` y abrieras la conexión con un `Connection conn = DriverManager.getConnection(...)` normal, sin cerrarla nunca, ¿qué pasaría si ejecutaras este `main()` 200 veces seguidas? Relaciona tu respuesta con el *pool* de conexiones que viste en el apartado de teoría de la semana 3.

---

## Mini-reto — segunda consulta

Repite exactamente el mismo patrón del Paso 1 (conexión, `PreparedStatement` parametrizado, recorrido del `ResultSet`, cierre con `try-with-resources`) para esta consulta nueva:

```sql
SELECT id, titulo, precio FROM videojuego WHERE precio < ?
```

Es decir: todos los videojuegos con precio inferior a un valor que pases como parámetro. Solo se te da el SQL objetivo — la estructura del código ya la tienes escrita en el Paso 1.

---

## Paso 3 — JDBC puro vs. `VideojuegoRepository`, con datos delante

Abre `VideojuegoRepository.java` de tu propio proyecto (una interfaz vacía que extiende `JpaRepository`) y compárala con lo que acabas de escribir.

**Responde**, con la comparación delante:

1. ¿Cuántas líneas de código hacen falta para "consultar todos los videojuegos de un estudio" con JDBC puro (cuenta las tuyas del Paso 1), frente a las que hacen falta con Spring Data JPA (una `Specification`, o incluso un simple método derivado por nombre como `findByEstudioId`)?
2. ¿Qué controlas con JDBC puro que no controlas igual de fino con JPA (pista: el SQL exacto que se ejecuta, sin que Hibernate decida nada por ti)?
3. ¿Qué se te olvida más fácilmente con JDBC puro que con JPA (pista: todo lo relacionado con abrir/cerrar recursos)?

---

## Paso 4 — Ver la inyección SQL de verdad, no solo leer sobre ella

!!! warning "Esto es una demostración deliberada de un fallo de seguridad"
    Vas a escribir código vulnerable a propósito, en tu clase de diagnóstico, solo para verlo pasar con tus propios ojos. Nunca hagas esto en código que vaya a producción.

Añade este método a tu clase `DiagnosticoJdbc` (o pruébalo sustituyendo temporalmente el `main`):

```java
private static void consultaInsegura(String tituloBuscado) throws SQLException {
    String sql = "SELECT id, titulo FROM videojuego WHERE titulo = '" + tituloBuscado + "'";
    System.out.println("SQL ejecutado: " + sql);

    try (Connection conn = DriverManager.getConnection(URL, USER, PASSWORD);
         Statement stmt = conn.createStatement();
         ResultSet rs = stmt.executeQuery(sql)) {

        while (rs.next()) {
            System.out.println("#" + rs.getLong("id") + " - " + rs.getString("titulo"));
        }
    }
}
```

Llámalo primero con un título normal (por ejemplo, `"Hades"`) — debería comportarse como esperas. Ahora llámalo con esta entrada exacta:

```java
consultaInsegura("' OR '1'='1");
```

**Observa** el SQL impreso por consola antes de ejecutarse, y el resultado que devuelve.

**Preguntas**:

1. ¿Qué SQL se ha ejecutado realmente (cópialo de la consola)? ¿Sigue teniendo la forma "buscar por un título concreto"?
2. ¿Cuántas filas ha devuelto, y por qué esa entrada concreta ha conseguido que se devuelvan todas, no solo las que coinciden con un título?
3. Vuelve al código del Paso 1: ¿por qué ese mismo ataque, probado contra el `PreparedStatement` con `?`, no funcionaría?

---

## ✅ Cierre

Has visto, con código propio y datos reales, la capa que Spring Data JPA te ha estado ocultando desde la Actividad 1.1 — y por qué esa capa existe: no solo por comodidad, también por seguridad (los parámetros de `PreparedStatement`). En la próxima actividad trabajas con procedimientos almacenados, invocados desde Java con `JdbcTemplate` — un punto intermedio entre lo que has visto hoy y la comodidad de JPA.
