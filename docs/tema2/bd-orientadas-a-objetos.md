<a id="bd-orientadas-a-objetos"></a>

# 🧩 3. Bases de datos orientadas a objetos, pruebas y documentación

Este apartado tiene dos partes bien diferenciadas: primero un bloque teórico sobre una categoría de bases de datos que no vas a usar en las prácticas (contenido puramente conceptual, exigido por el currículo), y después las pruebas que dan por buenas todas las piezas construidas hasta ahora.

---

## 🧊 Bases de datos orientadas a objetos puras

Una **base de datos orientada a objetos pura** va un paso más allá que lo objeto-relacional que has trabajado con JSONB: los objetos se persisten **tal cual**, con su identidad y sus referencias intactas, sin traducirlos a filas y columnas ni siquiera parcialmente. Recuerda la diferencia con JSONB: allí, una columna de una tabla relacional normal contenía un objeto JSON — seguía siendo, en el fondo, una tabla con filas. En una BD orientada a objetos pura no hay tablas en absoluto: el propio motor entiende y almacena objetos directamente.

Esto tiene una consecuencia directa: **no hace falta un ORM**. Un ORM (Tema 1) existe precisamente para traducir entre el mundo de objetos de tu programa y el mundo de tablas de una base de datos relacional — si el motor ya entiende objetos de forma nativa, ese puente deja de ser necesario.

El lenguaje de consulta asociado a este modelo se llama **OQL** (*Object Query Language*) — comparado con JPQL (Tema 1), que opera sobre entidades y propiedades pero sigue traduciéndose a SQL relacional por debajo, OQL consulta directamente sobre el modelo de objetos nativo del motor, sin esa traducción intermedia.

Algunos gestores históricos/conceptuales de esta categoría: **db4o**, **ObjectDB**, **Versant**. Los mencionas aquí solo como referencia de que existen — no vas a instalar ninguno, este bloque es puramente conceptual.

---

## 🧪 Pruebas y documentación

De vuelta al código: cómo se prueba y documenta lo construido en este tema.

### Test de capa (service y controller)

Ya conoces la distinción del Tema 1 de PSP (aunque aquí es Acceso a Datos, la idea es la misma): un test de **service** aísla la lógica de negocio, mockeando el repositorio; un test de **controller** (con MockMvc) prueba la capa HTTP, mockeando el service. Cada uno prueba una capa distinta, con distintos colaboradores mockeados — y ambos aplican igual cuando la entidad lleva una columna JSONB.

### Test de integración real (con Testcontainers)

```java
@Testcontainers
@AutoConfigureMockMvc
class LibreriaApiTest {

    @Container
    @ServiceConnection
    static PostgreSQLContainer<?> postgres = new PostgreSQLContainer<>("postgres:16-alpine");

    @Autowired
    private MockMvc mockMvc;

    // ...
}
```

`@Testcontainers` + `@Container` levantan un **PostgreSQL real** dentro de un contenedor Docker, solo para la duración del test — no una base de datos en memoria, no un mock. `@ServiceConnection` conecta automáticamente ese contenedor a tu aplicación de test, sin que configures manualmente la URL de conexión.

¿Por qué esto da más confianza que mockear el repositorio? Porque prueba el mapeo JSONB **real** contra un PostgreSQL **real** — una base de datos en memoria genérica (como H2) podría no soportar `jsonb_exists` exactamente igual, o ni siquiera soportar el tipo `jsonb` de PostgreSQL. Un test con Testcontainers detectaría un error de mapeo que un test con mocks jamás vería, porque el mock nunca ejecuta SQL de verdad contra ningún motor.

### Documentar, en el sentido de este RA

"Documentar" aquí no significa escribir un documento externo aparte — significa que el propio código de test, con nombres de método descriptivos (`crearLibro_DebeGuardarDetallesEdicion_CuandoEsValido`, por ejemplo) y comentarios donde haga falta, deja claro **qué** comportamiento se está verificando y **por qué**. El test bien escrito es, en sí mismo, la documentación de qué se espera que haga el sistema.

---

## ✅ Ideas clave

??? tip "Abrir resumen"

    - Una BD orientada a objetos **pura** persiste objetos tal cual, sin traducirlos a filas/columnas — por eso no necesita ORM.
    - **OQL** consulta directamente sobre el modelo de objetos nativo, sin traducción a SQL relacional (a diferencia de JPQL).
    - Un test de **service** aísla lógica (mock del repositorio); un test de **controller** prueba la capa HTTP (mock del service); un test de **integración** con Testcontainers levanta un motor real en Docker.
    - Testcontainers detecta errores de mapeo real (como con `jsonb`) que un mock nunca podría detectar.
    - "Documentar" en este RA es, sobre todo, que el propio test (nombre + estructura) deje claro qué se verifica y por qué.
