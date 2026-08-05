# 🧪 Actividad 4.3: Integración final del proyecto

!!! warning "Descarga la plantilla"
    📄 [Plantilla 4.3 — Integración final del proyecto](plantillas/Actividad_4_3_AD_Plantilla.docx){target="_blank" rel="noopener"}

!!! success "Entrega final del módulo"
    Esta es la última actividad evaluable de Acceso a Datos. No es una tarea más — es el cierre de todo lo que has construido desde el Tema 1: catálogo en PostgreSQL, reseñas en MongoDB, componentes desacoplados, y todas las mejoras que has ido añadiendo por el camino.

## Qué vas a entregar

- Un test de integración de extremo a extremo, con Testcontainers, que recorre tu GameVault completo.
- Una captura de ese test, y el resto de la clase, en verde desde tu IDE.
- Una captura de tu pipeline de CI ejecutando todos los tests en verde, tras ampliar su filtro.

---

## Requisitos previos

Tu GameVault completo: catálogo (JDBC, JPA, JSONB), reviews (MongoDB), y todos los componentes de este tema.

---

## Paso 1 — El test de integración de flujo completo

No partes de cero: tu `VideojuegoApiIntegrationTest` (Actividad 2.3) ya tiene los tres motores reales conectados —PostgreSQL desde el principio, MongoDB desde la Actividad 4.2, RabbitMQ desde la Actividad 3.1 de PSP— y ya tiene `loginComoAdmin()` y un helper para crear un estudio. No vas a crear ninguna clase nueva ni ningún `@Container` nuevo: añades un `@Test` más a esa misma clase, con esta forma:

```java
@Test
void flujoCompletoDeCatalogoResenasYBorradoEnCascada() throws Exception {
    String adminToken = loginComoAdmin();
    Long estudioId = crearEstudio(adminToken);

    // Tramo 1: crea el videojuego, crea una reseña, comprueba el resumen

    // Tramo 2: borra el videojuego, comprueba que la reseña ha desaparecido

    // Tramo 3: comprueba que el borrado también ha quedado en el registro de actividad
}
```

Cada tramo encadena sobre el mismo `videojuegoId`, así que declara esa variable en cuanto la tengas y reutilízala en los tres.

### Tramo 1 — la reseña y el resumen

Crea el videojuego reutilizando el mismo patrón de `POST /api/v1/videojuegos` con `detallesPlataforma` que ya ves en los tests existentes de la clase, y crea la reseña con un `ReviewRequestDTO` (`puntuacion`, `comentario`) — esas dos peticiones las escribes tú, siguiendo el mismo patrón que el resto de la clase. La comprobación del resumen sí te la doy hecha:

```java
// crea el videojuego (POST /api/v1/videojuegos) y guarda su videojuegoId

// crea la reseña (POST /api/v1/videojuegos/{videojuegoId}/reviews, con un ReviewRequestDTO)

mockMvc.perform(get("/api/v1/videojuegos/" + videojuegoId + "/reviews/resumen"))
        .andExpect(status().isOk())
        .andExpect(jsonPath("$.totalReviews").value(1))
        .andExpect(jsonPath("$.puntuacionMedia").value(8.0)); // el valor que le hayas dado a tu reseña
```

### Tramo 2 — el borrado en cascada

Borra el videojuego (`DELETE /api/v1/videojuegos/{videojuegoId}`, con `adminToken`, exige rol `ADMIN`) y verifica el borrado en cascada de la Actividad 3.2. El `RabbitMQContainer` ya entrega el mensaje de verdad, pero sigue siendo asíncrono: entre que borras y que el listener procesa el evento pasa un instante, así que hace falta un margen de espera antes de comprobar que la reseña ya no está — este proyecto no tiene ninguna librería de espera activa (como Awaitility) entre sus dependencias, así que un `Thread.sleep(...)` sencillo es suficiente:

```java
// borra el videojuego (DELETE /api/v1/videojuegos/{videojuegoId}, con adminToken)

Thread.sleep(1000); // margen para que el listener asíncrono procese el borrado

assertTrue(reviewRepository.findByVideojuegoId(videojuegoId).isEmpty());
```

!!! warning "¿Por qué no vale `GET .../reviews` para esto?"
    Es la comprobación más intuitiva —el mismo endpoint que ya usas para leer reseñas—, pero aquí falla: `ReviewService.findByVideojuegoId` empieza comprobando que el videojuego existe, y como acabas de borrarlo, esa comprobación devuelve 404 antes incluso de mirar si quedan reseñas. No es un fallo del borrado en cascada, es que estás preguntando por reseñas de un videojuego que ya no está. Para comprobar la cascada de verdad, necesitas ir directo al repositorio: añade `private ReviewRepository reviewRepository;` como campo `@Autowired` de la clase (mismo patrón que `mockMvc`), con su import correspondiente.

### Tramo 3 — un evento, dos colas

El borrado que acabas de disparar no tiene un único destino: sobre el mismo exchange (`RabbitMQConfig`, Actividad 3.1 de PSP) hay **dos** colas independientes escuchando `videojuego.eliminado` — la del borrado en cascada de reseñas, que acabas de comprobar, y la del registro de actividad. Amplía el mismo test para comprobar también la segunda:

```java
mockMvc.perform(get("/api/v1/actividad")
                .header("Authorization", "Bearer " + adminToken))
        .andExpect(status().isOk())
        .andExpect(jsonPath("$[0].tipo").value("VIDEOJUEGO_ELIMINADO"))
        .andExpect(jsonPath("$[0].entidadId").value(videojuegoId.toString()));
```

`GET /api/v1/actividad` devuelve una lista de `ActividadResponseDTO` ordenada de más reciente a más antiguo, así que el borrado que acabas de hacer es el primer elemento — un array plano (`$[0]`), a diferencia del `$.content[0]` que ya conoces de las listas paginadas. Fíjate en que `entidadId` se compara como `String`, no como número.

**Pregunta**: si solo comprobaras una de las dos colas, ¿qué tipo de fallo de configuración —piensa en el `binding`/`routing key` de `RabbitMQConfig`— podría pasar completamente desapercibido? Relaciónalo con la predicción que has hecho en la teoría de este apartado.

**Captura**: `flujoCompletoDeCatalogoResenasYBorradoEnCascada`, y el resto de tests de `VideojuegoApiIntegrationTest`, en verde, ejecutados desde tu IDE.

---

## Paso 2 — Amplía el filtro del CI, y verifica

Antes de comprobar nada, hay algo que arreglar en `.github/workflows/ci.yml`. Ese workflow lo has configurado en la Actividad 1.3 de PSP, filtrando los tests con `-Dtest='*ControllerTest'` porque en aquel momento el único test "pesado" del proyecto, `GamevaultApplicationTests`, no podía correr en CI. Desde entonces has construido muchos más tipos de test que ese filtro nunca ha recogido — de integración con Testcontainers (Actividad 2.3, y el que acabas de terminar aquí mismo) y de componente aislado con Mockito (Actividad 4.1, 4.2) — ninguno se ha estado ejecutando en tu CI hasta hoy, porque ninguno termina en `ControllerTest`.

!!! tip "Antes de tocar el filtro, mira qué es `GamevaultApplicationTests`"
    Ábrelo: es el `contextLoads()` vacío que genera Spring Initializr por defecto, contra el perfil `@ActiveProfiles("dev")` — de ahí que necesite una PostgreSQL, MongoDB y RabbitMQ reales que el runner no tiene. Pero tu `VideojuegoApiIntegrationTest` (Paso 1) ya arranca ese mismo contexto completo, con Testcontainers en vez de un perfil que el runner no puede satisfacer. Ya no aporta ninguna cobertura que el otro test no tenga: solo repite la misma pregunta ("¿arranca la aplicación?"), peor.

**Bórralo.** Con esa clase fuera, ya no queda ningún test que no pueda ejecutarse en cualquier entorno, así que el filtro entero deja de hacer falta:

```yaml
      - name: Ejecutar los tests
        run: ./mvnw test -B
```

Sin `-Dtest`, Maven Surefire ejecuta todo lo que encuentra —controller, integración, componente aislado, y cualquier test nuevo que construyas después— sin que tengas que tocar el YAML otra vez. La complejidad que aquella nota de PSP 1.3 dejaba fuera de alcance (levantar Postgres a mano dentro del workflow, con un servicio de GitHub Actions) nunca ha llegado a hacer falta: Testcontainers la resuelve sola, porque `ubuntu-latest` ya trae Docker instalado.

Haz `push` de tus cambios y comprueba, en la pestaña **Actions** de tu repositorio GitHub, que el pipeline ejecuta correctamente todos tus tests — de controller, de integración y de componente aislado. **Captura**: el resultado en verde del workflow.

---

## ✅ Cierre del módulo

Con esta entrega se cierra Acceso a Datos entero. Tu GameVault ha evolucionado, actividad a actividad, desde un proyecto vacío hasta una aplicación con persistencia relacional, ORM, objeto-relacional, documental y componentes desacoplados — probado de extremo a extremo y verificado automáticamente en cada cambio. Buen trabajo.
