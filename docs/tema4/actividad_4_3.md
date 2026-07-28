# 🧪 Actividad 4.3: Integración final del proyecto

!!! success "Entrega final del módulo"
    Esta es la última actividad evaluable de Acceso a Datos. No es una tarea más — es el cierre de todo lo que has construido desde el Tema 1: catálogo en PostgreSQL, reseñas en MongoDB, componentes desacoplados, y todas las mejoras que has ido añadiendo por el camino.

## Qué vas a entregar

- Un test de integración de extremo a extremo, con Testcontainers, que recorre tu GameVault completo.
- Un documento breve que resuma los componentes que has construido.
- Una autoevaluación personal y razonada.

---

## Requisitos previos

Tu GameVault completo: catálogo (JDBC, JPA, JSONB), reviews (MongoDB), y todos los componentes de este tema.

---

## Paso 1 — El test de integración de flujo completo

### Tramo 1 — guiado al completo

```java
package com.tunombre.gamevault.integration;

import com.jayway.jsonpath.JsonPath;
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.boot.testcontainers.service.connection.ServiceConnection;
import org.springframework.http.MediaType;
import org.springframework.test.context.ActiveProfiles;
import org.springframework.test.web.servlet.MockMvc;
import org.springframework.boot.webmvc.test.autoconfigure.AutoConfigureMockMvc;
import org.testcontainers.containers.PostgreSQLContainer;
import org.testcontainers.mongodb.MongoDBContainer;
import org.testcontainers.junit.jupiter.Container;
import org.testcontainers.junit.jupiter.Testcontainers;

import static org.springframework.test.web.servlet.request.MockMvcRequestBuilders.*;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.*;

@SpringBootTest
@AutoConfigureMockMvc
@ActiveProfiles("test")
@Testcontainers
class FlujoCompletoIntegrationTest {

    @Container
    @ServiceConnection
    static PostgreSQLContainer<?> postgres = new PostgreSQLContainer<>("postgres:16-alpine");

    @Container
    @ServiceConnection
    static MongoDBContainer mongodb = new MongoDBContainer("mongo:7");

    @Autowired
    private MockMvc mockMvc;

    private String loginComoAdmin() throws Exception {
        String respuesta = mockMvc.perform(post("/api/v1/auth/login")
                        .contentType(MediaType.APPLICATION_JSON)
                        .content("""
                                {
                                  "username": "admin",
                                  "password": "admin123"
                                }
                                """))
                .andExpect(status().isOk())
                .andReturn().getResponse().getContentAsString();

        return JsonPath.read(respuesta, "$.accessToken");
    }

    @Test
    void flujoCompleto_DesdeCrearHastaBorrarConReviews() throws Exception {
        String adminToken = loginComoAdmin();

        // Tramo 1: crear un estudio y un videojuego con detallesPlataforma
        mockMvc.perform(post("/api/v1/estudios")
                        .header("Authorization", "Bearer " + adminToken)
                        .contentType(MediaType.APPLICATION_JSON)
                        .content("""
                                {"nombre": "Team Cherry", "pais": "Australia"}
                                """))
                .andExpect(status().isCreated());

        String respuestaVideojuego = mockMvc.perform(post("/api/v1/videojuegos")
                        .header("Authorization", "Bearer " + adminToken)
                        .contentType(MediaType.APPLICATION_JSON)
                        .content("""
                                {
                                  "titulo": "Hollow Knight",
                                  "precio": 14.99,
                                  "fechaLanzamiento": "2017-02-24",
                                  "estudioId": 1,
                                  "detallesPlataforma": {"steam": {"idApp": 367520}}
                                }
                                """))
                .andExpect(status().isCreated())
                .andReturn().getResponse().getContentAsString();

        // extrae el id del videojuego creado (con JsonPath, por ejemplo) para los tramos siguientes
    }
}
```

Este primer tramo crea el estudio y el videojuego que vas a usar en el resto del test — con `@ServiceConnection` en dos contenedores a la vez, PostgreSQL y MongoDB corriendo de verdad, simultáneamente. Fíjate en `loginComoAdmin()` y en `@ActiveProfiles("test")`: los dos son exactamente el mismo patrón que ya has construido en la Actividad 2.3 (incluido tu propio `application-test.yml`, que reutilizas tal cual) — sin login, estos `POST` devolverían `403`, no `201`, porque `/api/v1/estudios` y `/api/v1/videojuegos` exigen rol `ADMIN` desde la Actividad 2.5 de PSP.

### Tramo 2 — mini-reto: la reseña y el resumen

Sin más código dado, amplía el mismo test (`@Test`) para: crear una reseña para ese videojuego (reutiliza el mismo `adminToken` — crear una reseña solo exige estar autenticado, no rol `ADMIN`, así que te sirve igual), y comprobar con un `GET .../resumen` que el total y la puntuación media son correctos. Repite el patrón que ya has usado en la Actividad 2.3 (MockMvc completo, `jsonPath` sobre el cuerpo).

### Tramo 3 — mini-reto: el borrado en cascada

Borra el videojuego (de nuevo con `adminToken`, exige rol `ADMIN`), y verifica el borrado en cascada de la Actividad 3.2: la reseña que has creado en el Tramo 2 debe desaparecer de MongoDB tras un pequeño margen de espera (recuerda: es asíncrono, vía RabbitMQ — puede que necesites un `Thread.sleep` breve o un mecanismo de espera activa en el test, ya que Testcontainers no siempre incluye RabbitMQ salvo que añadas también ese contenedor).

---

## Paso 2 — Documento de cierre

Escribe un documento breve (README de tu propio GameVault, o una sección de memoria si tu centro lo pide así) que liste los componentes que has desarrollado, con 1-2 frases por componente describiendo su responsabilidad. No es un manual exhaustivo, es un mapa rápido de "qué hace cada pieza".

Como mínimo, incluye: `CatalogoConsultaService`, `ReviewsConsultaService`, el flujo de borrado en cascada, el procedimiento almacenado `ajustar_precio_estudio`, el ranking JPQL de estudios.

---

## Paso 3 — Autoevaluación razonada

De todos los temas trabajados en este módulo, ¿con cuál te sientes más flojo, y por qué concretamente? No una respuesta genérica ("necesito practicar más") — señala una pieza técnica exacta con la que todavía no te sientes cómodo, y qué harías para reforzarla.

---

## Paso 4 — Amplía el filtro del CI, y verifica

Antes de comprobar nada, hay algo que arreglar en `.github/workflows/ci.yml`. Ese workflow lo configuraste en la Actividad 1.3 de PSP, y desde entonces filtra los tests con `-Dtest='*ControllerTest'` — una decisión de aquel momento, cuando el único test "pesado" del proyecto era `GamevaultApplicationTests` (el de arranque completo, sin ninguna PostgreSQL real disponible en el runner). Desde entonces has construido tests de integración reales con Testcontainers — el de la Actividad 2.3 de AD, y el que acabas de terminar aquí mismo — pero ese filtro nunca se amplió, así que ninguno de los dos se ha estado ejecutando en tu CI hasta hoy.

La buena noticia: no hace falta la complejidad que aquella nota de PSP 1.3 anticipaba (levantar servicios manuales en el propio workflow) — los runners de GitHub Actions (`ubuntu-latest`) ya traen Docker instalado, así que Testcontainers puede levantar sus propios contenedores sin tocar el YAML más allá del filtro. Amplíalo para que incluya también tus tests de integración, sin arrastrar `GamevaultApplicationTests` (que seguiría sin poder pasar en CI, al no usar Testcontainers):

```yaml
      - name: Ejecutar los tests
        run: ./mvnw test -B -Dtest='*ControllerTest,*IntegrationTest'
```

Haz `push` de tus cambios y comprueba, en la pestaña **Actions** de tu repositorio GitHub, que el pipeline ejecuta correctamente estos tests de integración. **Captura**: el resultado en verde del workflow.

---

## ✅ Cierre del módulo

Con esta entrega se cierra Acceso a Datos entero. Tu GameVault ha evolucionado, actividad a actividad, desde un proyecto vacío hasta una aplicación con persistencia relacional, ORM, objeto-relacional, documental y componentes desacoplados — probado de extremo a extremo y verificado automáticamente en cada cambio. Buen trabajo.
