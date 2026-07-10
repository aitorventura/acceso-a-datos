<a id="procedimientos-almacenados"></a>

# 🧩 5. Procedimientos almacenados

Añades una pieza distinta a todo lo visto hasta ahora: código que no vive en tu aplicación Java, sino dentro del propio motor de base de datos.

---

## 🗄️ Qué es un procedimiento almacenado

Un **procedimiento almacenado** es código SQL (con lógica: variables, condiciones, bucles) que se guarda y se ejecuta dentro del propio gestor de base de datos, no en tu aplicación. Lo defines una vez, y a partir de ahí se invoca por su nombre — el motor ejecuta esa lógica cerca de los datos, sin que viajen por la red hacia tu aplicación y vuelvan.

Cada gestor tiene su propio lenguaje procedural para escribirlos — en PostgreSQL es **PL/pgSQL**, una extensión de SQL con variables, parámetros y control de flujo:

```sql
CREATE OR REPLACE PROCEDURE nombre_procedimiento(parametro TIPO)
LANGUAGE plpgsql
AS $$
BEGIN
    -- lógica con SQL normal aquí dentro
END;
$$;
```

!!! tip "Procedimiento vs. función"
    Ambos son código guardado en el gestor, pero una **función** siempre devuelve un valor y se puede usar dentro de una consulta (`SELECT mi_funcion(x)`); un **procedimiento** no tiene por qué devolver nada y se invoca de forma independiente con `CALL`. Cuando lo que quieres es "ejecutar una acción" más que "calcular un valor", un procedimiento encaja mejor.

### Cuándo tiene sentido usarlos — y cuándo no

Tienen sentido cuando una operación conviene que ocurra **atómicamente, cerca de los datos** — por ejemplo, un ajuste masivo de precios que afecta a muchas filas a la vez: hacerlo dentro del motor evita traer todas las filas a Java, modificarlas una a una, y devolverlas, con todo el tráfico de red que eso implica.

El inconveniente: la lógica queda repartida en dos sitios (tu código Java y la base de datos), lo que la hace más difícil de versionar junto con el resto del proyecto y de probar con las mismas herramientas que usas para el resto de tu código.

---

## ☎️ Cómo se invocan desde Java

La vía estándar de JDBC es `CallableStatement` (el primo de `PreparedStatement`, pensado específicamente para invocar procedimientos) — lo mencionas para que sepas que existe, pero GameVault usa una vía más cómoda: **`JdbcTemplate`**.

`JdbcTemplate` es el ayudante que trae Spring sobre JDBC puro: te evita exactamente la fontanería que viste el apartado anterior (abrir conexión, cerrar recursos, gestionar excepciones a mano) sin llegar a ser un ORM completo como el que verás en el Tema 2.

```java
jdbcTemplate.update("CALL ajustar_precio_estudio(?, ?)", estudioId, porcentaje);
```

Una sola línea: `JdbcTemplate` abre la conexión, prepara la sentencia con parámetros (igual de seguros frente a inyección SQL que el `PreparedStatement` que ya conoces), la ejecuta y cierra todo por ti.

---

## 🎮 Aterrizaje en GameVault: `ajustar_precio_estudio`

Esta es una pieza que todavía no existe en tu proyecto: ahora mismo tu `EstudioRepository.java` es un `JpaRepository` plano, sin ningún procedimiento almacenado ni uso de `JdbcTemplate`. Vas a añadirla tú, siguiendo el mismo estilo del resto de servicios del paquete `catalogo`.

### 1. El procedimiento en PostgreSQL

```sql
CREATE OR REPLACE PROCEDURE ajustar_precio_estudio(p_estudio_id BIGINT, p_porcentaje NUMERIC)
LANGUAGE plpgsql
AS $$
BEGIN
    UPDATE videojuego
    SET precio = ROUND(precio * (1 + p_porcentaje / 100.0), 2)
    WHERE estudio_id = p_estudio_id;
END;
$$;
```

Sube o baja, de una sola vez, el precio de **todos** los videojuegos de un estudio en el porcentaje indicado. `ROUND(..., 2)` importa: `Videojuego.precio` es un `BigDecimal` con precisión `10,2` en la base de datos — sin redondear explícitamente a dos decimales, el cálculo (`precio * (1 + porcentaje/100)`) podría dejar más decimales de los que la columna admite con precisión limpia.

### 2. La invocación desde el service

```java
@Service
@RequiredArgsConstructor
public class EstudioService {
    private final EstudioRepository estudioRepository;
    private final JdbcTemplate jdbcTemplate;

    public void ajustarPrecio(Long estudioId, BigDecimal porcentaje) {
        if (!estudioRepository.existsById(estudioId)) {
            throw new ResponseStatusException(HttpStatus.NOT_FOUND, "Estudio no encontrado");
        }
        jdbcTemplate.update("CALL ajustar_precio_estudio(?, ?)", estudioId, porcentaje);
    }
}
```

`JdbcTemplate` se inyecta exactamente igual que cualquier otro bean — junto al `EstudioRepository`, con `@RequiredArgsConstructor`, sin ninguna configuración especial (Spring Boot lo configura automáticamente en cuanto detecta un `DataSource` en el classpath, que ya tienes desde el Tema 1). La comprobación de "estudio no encontrado" la sigues haciendo con Spring Data JPA (`existsById`) antes de invocar el procedimiento — no hace falta que todo pase por `JdbcTemplate`, solo la parte que de verdad conviene ejecutar cerca de los datos.

### 3. Por qué procedimiento y no un bucle en Java

Podrías haber escrito esto en Java: cargar todos los videojuegos del estudio, recorrerlos con un bucle, modificar el precio de cada uno, y guardar. Funcionaría — pero el procedimiento almacenado lo hace en **una sola operación atómica** dentro del motor, sin traer ninguna fila a Java ni hacer múltiples viajes de red por cada videojuego. Para una actualización masiva como esta, es la opción más directa.

---

## 🧭 Recapitulación del tema

Con esto se completa el recorrido: por qué existen los conectores y cómo se define la estructura de la base de datos (apartado 2) → CRUD completo y transacciones gestionadas por Spring (apartado 3) → la misma fontanería, pero a mano, con JDBC puro (apartado 4) → código que vive dentro del propio motor, invocado desde Java (este apartado). El Tema 2 retoma exactamente donde empezó todo: el desfase objeto-relacional, ahora resuelto con una herramienta ORM completa (Hibernate) en vez de conectores manuales.

---

## ✅ Ideas clave

??? tip "Abrir resumen"

    - Un **procedimiento almacenado** es código con lógica que vive y se ejecuta dentro del gestor de base de datos, no en la aplicación.
    - PostgreSQL usa **PL/pgSQL** como lenguaje procedural; un procedimiento se invoca con `CALL`, una función se puede usar dentro de un `SELECT`.
    - Conviene usarlos para operaciones que deben ocurrir atómicamente y cerca de los datos (ajustes masivos); tienen el coste de repartir la lógica entre dos sitios.
    - **`JdbcTemplate`** es el ayudante de Spring sobre JDBC puro — invoca procedimientos con parámetros seguros (`jdbcTemplate.update("CALL ...", ...)`) sin la fontanería manual de abrir/cerrar recursos.
    - Se inyecta como cualquier otro bean, junto a los repositorios de Spring Data JPA — ambos enfoques conviven en el mismo service.
