# 🧪 Actividad 4.2: Mismo patrón, otro motor — un componente sobre el módulo documental

!!! warning "Descarga la plantilla"
    📄 [Plantilla 4.2 — Mismo patrón, otro motor: un componente sobre el módulo documental](plantillas/Actividad_4_2_AD_Plantilla.docx){target="_blank" rel="noopener"}

!!! info "Menos código dado que en la 4.1 — la estructura ya la conoces"
    Hoy repites, sobre MongoDB, el mismo patrón exacto de componente que ya has construido sobre PostgreSQL en la Actividad 4.1. Como ya has pasado por esto una vez, aquí encuentras menos código ya escrito y más guía: la estructura la conoces, así que el peso recae en que la apliques tú.

## Qué vas a practicar

- Replicar el patrón interfaz + implementación oculta sobre un motor distinto.
- Usar un componente nuevo desde otro módulo, sin conocer su implementación.
- Comparar dos componentes que resuelven el mismo problema sobre motores distintos.

---

## Requisitos previos

Tu `CatalogoConsultaService` (Actividad 4.1) como referencia visual constante.

---

## Paso 1 — Planteamiento: el contrato mínimo

¿Qué necesitaría saber el módulo `catalogo` (u otro futuro) sobre las reseñas, sin conocer nada de MongoDB? Define tú mismo el contrato mínimo: dos métodos, uno que devuelva cuántas reseñas tiene un videojuego (`long`) y otro que devuelva su puntuación media (`double`), los dos recibiendo el `id` del videojuego. La regla de cálculo ya la conoces de `ReviewService.getResumenByVideojuegoId`; ahora vas a implementar esa misma idea detrás de un contrato específico para que otros módulos puedan consultarla sin conocer `ReviewRepository` ni MongoDB.

---

## Paso 2 — El componente completo

!!! warning "El paquete `reviews.api` no existe todavía en tu proyecto"
    A diferencia de `catalogo.api` (que ya tienes de la Actividad 4.1), aquí no hay ningún precedente — el primer paso es crear la carpeta nueva.

Estructura de ficheros esperada:

```
reviews/
├── api/
│   └── ReviewsConsultaService.java      ← interfaz (nueva)
├── ReviewsConsultaServiceImpl.java      ← implementación (nueva, package-private)
├── Review.java
├── ReviewRepository.java
└── ReviewService.java
```

Sin más código dado que la estructura de arriba y el contrato del Paso 1, crea:

1. La interfaz `ReviewsConsultaService` en el paquete nuevo `reviews.api` — por analogía exacta con `catalogo.api.CatalogoConsultaService`.
2. `ReviewsConsultaServiceImpl` (package-private, anotada `@Service`) en `reviews`, que implemente la interfaz reutilizando `ReviewRepository` — el código es idéntico en estructura al de `CatalogoConsultaServiceImpl` que ya tienes delante, solo cambia la lógica interna (usa `findByVideojuegoId` y calcula total/media como ya hace `getResumenByVideojuegoId`).

---

## Paso 3 — Usarlo desde `catalogo`

Sin código dado, ahora te toca a ti conectar las dos piezas — y esta vez usas **los dos** métodos del componente, no solo uno:

1. Añade a `VideojuegoResponseDTO` dos campos nuevos: `totalReviews` (tipo `long`) y `puntuacionMedia` (tipo `Double`), sin ninguna anotación de validación — son datos de solo lectura, igual que `nombreEstudio`.
2. En `VideojuegoService`, inyecta `ReviewsConsultaService` —la interfaz, nunca la implementación— junto al resto de dependencias del constructor.
3. Dentro de `mapToDTO`, llama a `totalReviewsDe(v.getId())` y a `puntuacionMediaDe(v.getId())`, y pasa los dos resultados como esos dos argumentos nuevos. Es el mismo mecanismo que ya usas para `nombreEstudio` (`v.getEstudio().getNombre()`), solo que estos dos datos no salen de `Videojuego`, salen del componente nuevo.

**Fíjate**: `catalogo` no importa nada de `reviews` salvo la interfaz del paquete `api` — ni conoce `Review`, ni `ReviewRepository`, ni cómo se calculan esos dos datos por dentro.

!!! warning "Esto rompe tu `VideojuegoControllerTest`"
    Igual que pasó con `detallesPlataforma` en la Actividad 2.1, añadir dos campos nuevos a `VideojuegoResponseDTO` rompe cualquier `new VideojuegoResponseDTO(...)` que siga usando el constructor de los seis campos antiguos. Localiza los **tres** sitios de `VideojuegoControllerTest` donde pasa esto, y añade un valor `long` y un valor `Double` cualquiera al final de cada uno — los valores concretos no importan, ninguno de esos tests comprueba nada sobre `totalReviews` ni `puntuacionMedia`, solo necesitan que el DTO compile.

!!! note "Un diseño deliberadamente sencillo"
    Tal como has definido el contrato, `mapToDTO` llama a dos métodos y cada uno consulta las reseñas del videojuego, así que puedes terminar haciendo dos accesos a MongoDB por cada videojuego que mapeas. Para esta actividad lo mantenemos así porque permite practicar claramente dos operaciones del componente. En una aplicación con muchos datos sería mejor exponer una única operación de resumen que devolviera ambos valores con una sola consulta o cálculo.

---

## Paso 4 — Verificación

```bash
curl http://localhost:8080/api/v1/videojuegos/1
```

Usa el `id` que quieras, `1` o cualquier otro — pero elige uno que tenga **al menos una reseña** ya creada: con `totalReviews = 0`, `puntuacionMedia` sale `0.0` por el propio `.orElse(0.0)`, y no se distingue de un fallo real en el cálculo. Si no tienes claro qué videojuego tiene reseñas, consúltalo primero con `GET /api/v1/videojuegos/{id}/reviews`.

**Comprueba**: que la respuesta incluye `totalReviews` y `puntuacionMedia`, y que los dos datos coinciden con lo que calcula `GET /api/v1/videojuegos/{id}/reviews/resumen` para el mismo videojuego — ambos deben aplicar la misma regla de cálculo.

**Captura**: la respuesta de `GET /videojuegos/{id}` (con reseñas de verdad) con `totalReviews` y `puntuacionMedia`, junto a la de `GET /reviews/resumen` mostrando los mismos valores.

!!! warning "`VideojuegoApiIntegrationTest` necesita MongoDB a partir de ahora"
    Antes de esta actividad, `VideojuegoApiIntegrationTest` ya necesitaba PostgreSQL y RabbitMQ, pero todavía no consultaba MongoDB. Desde el Paso 3, `mapToDTO` llama a `ReviewsConsultaService` en **cada** petición sobre `Videojuego`, así que el test necesita ahora también un contenedor de MongoDB real, o falla con un error de conexión en cuanto ejecutes cualquier test de la clase.

    Añade la dependencia a tu `pom.xml`, junto a las demás dependencias de Testcontainers que ya tienes:

    ```xml
    <dependency>
        <groupId>org.testcontainers</groupId>
        <artifactId>mongodb</artifactId>
        <scope>test</scope>
    </dependency>
    ```

    Y el contenedor, junto a los de `postgres` y `rabbitmq` que ya tienes en `VideojuegoApiIntegrationTest`:

    ```java
    @Container
    @ServiceConnection
    static MongoDBContainer mongodb = new MongoDBContainer("mongo:8");
    ```

    No olvides el `import org.testcontainers.containers.MongoDBContainer;`.

**Ejecuta también** tus tests existentes (`VideojuegoControllerTest`, `CatalogoConsultaServiceImplTest`, `VideojuegoApiIntegrationTest`) y comprueba que siguen pasando, ya con el cambio del Paso 3 aplicado.

**Captura**: tu batería de tests completa en verde.

---

## Paso 5 — Prueba del componente aislado

Crea `ReviewsConsultaServiceImplTest` en `src/test/java/com/tunombre/gamevault/reviews/ReviewsConsultaServiceImplTest.java`, dentro del mismo paquete `reviews` que `ReviewsConsultaServiceImpl`. Al ser la implementación *package-private*, el test necesita compartir paquete con ella para poder acceder a su tipo.


Con `ReviewsConsultaService` ya construido, integrado en `catalogo` y verificado de extremo a extremo, toca aislarlo: un test aislado, siguiendo exactamente el patrón de `CatalogoConsultaServiceImplTest` (`@Mock` sobre `ReviewRepository`, `@InjectMocks` sobre `ReviewsConsultaServiceImpl`). Cubre, como mínimo, estos tres casos:

1. `totalReviewsDe`: con el mock de `findByVideojuegoId` devolviendo una lista de varias reseñas, comprueba que el resultado coincide con el tamaño de la lista.
2. `puntuacionMediaDe`: con el mock devolviendo reseñas con puntuaciones conocidas, comprueba que la media calculada es la correcta.
3. `puntuacionMediaDe` cuando no hay ninguna reseña: con el mock devolviendo una lista vacía, comprueba que devuelve `0.0` en vez de lanzar una excepción — es el mismo `.orElse(0.0)` que ya usa `getResumenByVideojuegoId`.

**Captura**: tus tests de `ReviewsConsultaServiceImplTest` en verde.

---

## Reflexión de cierre — tabla comparativa

Rellena esta tabla con tu propia experiencia:

| | `CatalogoConsultaService` (PostgreSQL/JPA) | `ReviewsConsultaService` (MongoDB) |
|---|---|---|
| ¿Dónde vive la interfaz? | | |
| ¿La implementación es `public` o package-private? | | |
| ¿Qué repositorio inyecta por debajo? | | |
| ¿El consumidor sabe qué motor hay debajo? | | |

**Conclusión** (2-3 frases propias): ¿en qué se diferencian las implementaciones de ambos componentes? ¿En qué son idénticas sus interfaces vistas desde fuera? La respuesta esperada: el patrón de componente es independiente del motor de persistencia que hay por debajo — se replica con exactamente el mismo molde.

**Una pregunta más**: con `CatalogoConsultaService` (Actividad 4.1) y `ReviewsConsultaService` (hoy), `reviews` importa `catalogo.api` y `catalogo` importa `reviews.api`. Las interfaces evitan que cada módulo conozca la implementación interna del otro, pero no eliminan por completo la dependencia entre ambos.

¿Qué se ha desacoplado realmente y qué dependencia sigue existiendo? ¿Por qué esta relación funciona dentro de la estructura actual del proyecto, pero podría convertirse en un problema si algún día `catalogo` y `reviews` se separasen en módulos Maven independientes que tuvieran que depender uno del otro?

---

## ✅ Cierre

Tienes dos componentes, sobre dos motores completamente distintos, con el mismo patrón de diseño exacto. En la última actividad del módulo integras todo lo construido en un test final de extremo a extremo.
