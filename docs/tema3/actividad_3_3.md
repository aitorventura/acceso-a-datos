# 🧪 Actividad 3.3: Pruebas de integración sobre JSONB

!!! info "Práctica guiada"
    Hoy escribes un test de integración real, con Testcontainers, que verifica de verdad el comportamiento de la columna JSONB y el filtro `jsonb_exists` que construiste en las dos actividades anteriores.

## Qué vas a practicar

- Configurar un test con Testcontainers y `@ServiceConnection`.
- Escribir un test de integración completo, vía la API, sobre JSONB.
- Entender qué detecta un test así que un test con mocks nunca detectaría.

---

## Requisitos previos

Tu columna `detallesPlataforma` (Actividad 3.1) y el filtro `disponibleEnPlataforma` (Actividad 3.2), ambos funcionando.

---

## Paso 1 — Configuración del test, guiada al completo

Añade las dependencias de Testcontainers a tu `pom.xml` (si no las tienes ya de otra actividad):

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-testcontainers</artifactId>
    <scope>test</scope>
</dependency>
<dependency>
    <groupId>org.testcontainers</groupId>
    <artifactId>junit-jupiter</artifactId>
    <scope>test</scope>
</dependency>
<dependency>
    <groupId>org.testcontainers</groupId>
    <artifactId>postgresql</artifactId>
    <scope>test</scope>
</dependency>
```

Crea la clase de test:

```java
package com.tunombre.gamevault.integration;

import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.boot.testcontainers.service.connection.ServiceConnection;
import org.springframework.http.MediaType;
import org.springframework.test.context.ActiveProfiles;
import org.springframework.test.web.servlet.MockMvc;
import org.springframework.boot.webmvc.test.autoconfigure.AutoConfigureMockMvc;
import org.testcontainers.containers.PostgreSQLContainer;
import org.testcontainers.junit.jupiter.Container;
import org.testcontainers.junit.jupiter.Testcontainers;

import static org.springframework.test.web.servlet.request.MockMvcRequestBuilders.*;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.*;

@SpringBootTest
@AutoConfigureMockMvc
@ActiveProfiles("test")
@Testcontainers
class JsonbIntegrationTest {

    @Container
    @ServiceConnection
    static PostgreSQLContainer<?> postgres = new PostgreSQLContainer<>("postgres:16-alpine");

    @Autowired
    private MockMvc mockMvc;

    // los tests van aquí
}
```

`@ServiceConnection` es la pieza clave: conecta automáticamente el `PostgreSQLContainer` a tu aplicación de test, sin que tengas que configurar manualmente `spring.datasource.url` en ningún `application-test.yaml` — Spring Boot lo resuelve por ti. Fíjate en que la imagen es `postgres:16-alpine`, no la `18-alpine` de tu `docker-compose.yaml` de desarrollo — recuerda del Tema 1 que ambas versiones no tienen por qué coincidir, mientras sean compatibles con las características (como JSONB) que usa el proyecto. `@ActiveProfiles("test")` activa un perfil de test (créalo si no lo tienes, con `ddl-auto: create-drop` — aquí sí conviene, porque cada ejecución del test parte de una base de datos limpia).

---

## Paso 2 — Primer test, guiado al completo

```java
@Test
void filtrarPorPlataforma_DebeDevolverElVideojuego_CuandoLaTiene() throws Exception {
    mockMvc.perform(post("/api/v1/videojuegos")
                    .contentType(MediaType.APPLICATION_JSON)
                    .content("""
                            {
                              "titulo": "Hades",
                              "precio": 24.99,
                              "fechaLanzamiento": "2020-09-17",
                              "estudioId": 1,
                              "detallesPlataforma": {"steam": {"idApp": 123}}
                            }
                            """))
            .andExpect(status().isCreated());

    mockMvc.perform(get("/api/v1/videojuegos?plataforma=steam"))
            .andExpect(status().isOk())
            .andExpect(jsonPath("$.content[0].titulo").value("Hades"));
}
```

(ajusta `estudioId: 1` a un estudio que exista de verdad en tu base de datos de test — puede que necesites crearlo primero con otro `POST`, o cargar datos de prueba antes del test).

Línea a línea: primero un `POST` real, vía MockMvc, que crea un videojuego con una plataforma concreta en su `detallesPlataforma`; después un `GET` filtrado por esa misma plataforma, comprobando con `jsonPath` que el videojuego creado aparece en el resultado. Todo esto ocurre contra el PostgreSQL **real** del contenedor — no hay ningún mock de por medio.

---

## Mini-reto — el test negativo

Repite el mismo patrón del Paso 2, pero esta vez filtrando por una plataforma que el videojuego creado **no tiene** (por ejemplo, `?plataforma=xbox`), y comprueba que la lista de resultados **no** incluye ese videojuego. Solo se indica el objetivo — la estructura es idéntica a la del Paso 2.

---

## Pregunta de comprensión

¿Qué detectaría este test con Testcontainers que **no** detectaría un test equivalente con el repositorio mockeado? Piensa en un error concreto: por ejemplo, si escribieras mal el nombre de la función SQL en `criteriaBuilder.function("jsonb_exists", ...)` (una errata tipográfica), o si el tipo de columna generado no fuera realmente `jsonb`. ¿Un mock del repositorio detectaría ese error? ¿Por qué sí o por qué no?

---

## Resumen de tu evolución

Escribe un resumen propio (5-6 líneas) de cómo tu GameVault ha evolucionado a lo largo de las tres actividades de este tema: desde la columna JSONB básica (Actividad 3.1) hasta una consulta filtrada y probada de verdad con Testcontainers (esta actividad).

Incluye, como parte explícita de este entregable, 2-3 líneas explicando por qué JSONB no abre ni cierra una conexión propia: reutiliza la misma conexión JDBC/JPA ya gestionada por el framework desde el Tema 1.

---

## ✅ Cierre

En el Tema 4 das el salto a MongoDB: la primera base de datos NoSQL distinta de PostgreSQL de todo el curso, con un modelo documental nativo, sin tablas de ningún tipo.
