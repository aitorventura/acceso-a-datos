# 🧪 Actividad 1.4: Procedimiento almacenado con `JdbcTemplate`

!!! info "Práctica guiada"
    Hoy añades a tu GameVault el procedimiento almacenado `ajustar_precio_estudio`, lo invocas desde Java con `JdbcTemplate` y lo expones como un endpoint REST nuevo.

## Qué vas a practicar

- Crear un procedimiento almacenado en PostgreSQL.
- Invocarlo desde Spring con `JdbcTemplate`.
- Exponerlo como un endpoint REST siguiendo el estilo ya establecido del proyecto.

---

## Requisitos previos

Tu `EstudioController`/`EstudioService` de la Actividad 1.2 (con al menos `GET` y `POST`) y algún dato de prueba (un estudio con varios videojuegos asociados).

---

## Paso 1 — El procedimiento almacenado

Conéctate a tu PostgreSQL (psql, DBeaver o pgAdmin) y ejecuta:

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

Línea a línea: recibe el `id` de un estudio y un porcentaje; actualiza, de una sola vez, el precio de **todos** los videojuegos de ese estudio (`WHERE estudio_id = p_estudio_id`), multiplicándolo por `(1 + porcentaje/100)` y redondeando a 2 decimales con `ROUND` (recuerda: `precio` es un `BigDecimal` con precisión `10,2` en la base de datos — sin este redondeo explícito, el cálculo podría dejar decimales de más).

**Verificación rápida en SQL**, antes de tocar Java: comprueba los precios actuales de un estudio, invoca el procedimiento a mano, y vuelve a comprobarlos:

```sql
SELECT id, titulo, precio FROM videojuego WHERE estudio_id = 1;
CALL ajustar_precio_estudio(1, 10); -- sube un 10%
SELECT id, titulo, precio FROM videojuego WHERE estudio_id = 1;
```

---

## Paso 2 — Invocación desde el service, guiada al completo

Añade `JdbcTemplate` a `EstudioService`, junto al repositorio que ya tienes:

```java
@Service
@RequiredArgsConstructor
public class EstudioService {
    private final EstudioRepository estudioRepository;
    private final JdbcTemplate jdbcTemplate;

    // ... tus métodos ya existentes (findAll, create, update, delete) ...

    public void ajustarPrecio(Long estudioId, BigDecimal porcentaje) {
        if (!estudioRepository.existsById(estudioId)) {
            throw new ResponseStatusException(HttpStatus.NOT_FOUND, "Estudio no encontrado");
        }
        jdbcTemplate.update("CALL ajustar_precio_estudio(?, ?)", estudioId, porcentaje);
    }
}
```

No necesitas registrar `JdbcTemplate` en ningún sitio: Spring Boot lo configura automáticamente en cuanto detecta un `DataSource` en el classpath (el mismo que ya usa tu Spring Data JPA desde la Actividad 1.1) — solo tienes que declararlo como dependencia en el constructor, exactamente igual que `EstudioRepository`.

Crea un DTO mínimo para el porcentaje:

```java
public record AjustePrecioDTO(
        @NotNull @Digits(integer = 3, fraction = 2) BigDecimal porcentaje
) {}
```

Y el endpoint en `EstudioController`:

```java
@PostMapping("/{id}/ajustar-precio")
public ResponseEntity<Void> ajustarPrecio(@PathVariable Long id, @Valid @RequestBody AjustePrecioDTO dto) {
    estudioService.ajustarPrecio(id, dto.porcentaje());
    return ResponseEntity.noContent().build();
}
```

Prueba con `curl` o Swagger UI:

```bash
curl -X POST http://localhost:8080/api/v1/estudios/1/ajustar-precio \
  -H "Content-Type: application/json" \
  -d '{"porcentaje": -15}'
```

**Comprueba**: con un `GET /api/v1/videojuegos` (o consultando directamente en `psql`), que los precios de los videojuegos de ese estudio han bajado un 15%.

---

## Mini-reto — una salvaguarda contra precios negativos

Ahora mismo, si aplicas un porcentaje muy negativo (por ejemplo, `-150`), el procedimiento dejaría precios en negativo — algo que no tiene sentido de negocio. Modifica el `UPDATE` del procedimiento para que ningún precio resultante quede por debajo de `0`.

**Pista**: la función SQL `GREATEST(a, b)` devuelve el mayor de los dos valores — puedes envolver el cálculo del nuevo precio con ella para que nunca baje de `0`.

Vuelve a ejecutar `CREATE OR REPLACE PROCEDURE` con tu versión corregida (el `OR REPLACE` te permite redefinirlo sin borrarlo antes) y comprueba con un porcentaje muy negativo que ahora los precios se quedan en `0`, no en negativo.

---

## Pregunta final

Repasa, en 3-4 frases propias, cómo esta actividad conecta con las tres anteriores de este tema: empezaste entendiendo por qué existen los conectores y cómo se define la estructura de la base de datos (Actividad 1.1), construiste un CRUD completo con transacciones gestionadas por Spring (Actividad 1.2), viste la misma fontanería pero a mano con JDBC puro (Actividad 1.3), y hoy has ejecutado código que vive dentro del propio motor de base de datos. ¿Cuál de las cuatro piezas te ha parecido más cercana a "lo que ya sabías de SQL" y cuál más nueva del todo?

---

## ✅ Cierre

En el Tema 2 vuelves a la comodidad de Spring Data JPA, pero esta vez con Hibernate como herramienta ORM completa — vas a instalarla, configurarla y usarla para persistir objetos sin escribir SQL a mano, retomando exactamente el desfase objeto-relacional del que arrancó este tema.
