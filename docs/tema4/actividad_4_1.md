# 🧪 Actividad 4.1: `CatalogoConsultaService` — un componente reutilizable

!!! warning "Descarga la plantilla"
    📄 [Plantilla 4.1 — CatalogoConsultaService: un componente reutilizable](plantillas/Actividad_4_1_AD_Plantilla.docx){target="_blank" rel="noopener"}

!!! info "Práctica guiada"
    Desde la Actividad 3.1, `ReviewService` inyecta `VideojuegoRepository` directamente. Hoy resuelves ese acoplamiento construyendo tu primer componente con contrato explícito, siguiendo el patrón de la teoría, y lo extiendes con un método nuevo.

## Qué vas a practicar

- Construir un componente con interfaz separada de su implementación oculta.
- Razonar sobre las consecuencias del desacoplamiento con un caso concreto.
- Extender un componente propio siguiendo su propio patrón.

---

## Requisitos previos

Tu `ReviewService` de la Actividad 3.1, con `VideojuegoRepository` inyectado directamente — es exactamente el acoplamiento que vas a resolver hoy.

---

## Paso 1 — Construye el componente

Sin más código dado que el patrón de la teoría, crea:

1. La interfaz `CatalogoConsultaService`, en un paquete nuevo `catalogo.api`, con un único método por ahora: `boolean existeVideojuego(Long videojuegoId)`.
2. `CatalogoConsultaServiceImpl` (package-private, anotada `@Service`) en `catalogo`, que implemente la interfaz reutilizando tu `VideojuegoRepository` ya existente.
3. Modifica `ReviewService` para que inyecte `CatalogoConsultaService` (la interfaz) en lugar de `VideojuegoRepository` directamente, y actualiza **las cuatro** comprobaciones que hoy usan `videojuegoRepository.existsById(...)` — `findByVideojuegoId`, `create`, `getResumenByVideojuegoId` y `findBuenasNotas` — para que llamen a `existeVideojuego(...)`. Al terminar, ningún método de `ReviewService` debería seguir usando `videojuegoRepository` — quita también el campo y su `import`.

`ReviewRepository`, en cambio, no lo tocas: es el propio repositorio del módulo `reviews`, no el de otro módulo — inyectarlo directamente nunca ha sido el problema. El acoplamiento que resuelves hoy es solo el que cruza hacia `catalogo`.

Reinicia tu aplicación y comprueba que nada se ha roto — crea una reseña para un videojuego existente, y prueba también con un `videojuegoId` que no exista:

```bash
curl -X POST http://localhost:8080/api/v1/videojuegos/1/reviews \
  -H "Authorization: Bearer $USER_TOKEN" -H "Content-Type: application/json" \
  -d '{"puntuacion": 8, "comentario": "Sigue funcionando igual"}'
```

**Comprueba**: mismo comportamiento de siempre — `201 Created` si el videojuego existe, `404 Not Found` si no. El refactor no debería cambiar ni un código de estado, solo de dónde viene la comprobación.

**Captura**: la respuesta `201`, confirmando que el componente nuevo funciona igual que el `VideojuegoRepository` directo que sustituye.

---

## Paso 2 — Preguntas de comprensión

Con tu código recién escrito delante, responde:

1. ¿Por qué `CatalogoConsultaService` (la interfaz) vive en el paquete `catalogo.api`, y `CatalogoConsultaServiceImpl` no?
2. ¿Por qué `CatalogoConsultaServiceImpl` no lleva el modificador `public`? ¿Qué pasaría si lo añadieras — cambiaría algo en cómo funciona, o solo en qué es visible desde fuera del paquete?
3. En `ReviewService`, ¿qué tipo declaras al inyectar esta dependencia — la interfaz `CatalogoConsultaService`, o la clase `CatalogoConsultaServiceImpl`? ¿Podrías, siquiera, declarar el tipo concreto desde otro paquete?

---

## Paso 3 — El desacoplamiento, con un caso concreto

Imagina que mañana decides sustituir JPA por otra tecnología de persistencia para el catálogo (no lo vas a hacer, es un ejercicio de razonamiento). **Responde**:

- ¿Qué ficheros del paquete `catalogo` tendrías que tocar?
- ¿Tendrías que tocar algo de `reviews`?
- ¿Qué papel concreto juega la interfaz `CatalogoConsultaService` en que la respuesta a la pregunta anterior sea "no"?

---

## Paso 4 — Un segundo método

Sin más código dado que el patrón que ya tienes delante, añade un segundo método al componente: `String tituloDe(Long videojuegoId)`, que devuelva el título de un videojuego (o lance `ResponseStatusException(HttpStatus.NOT_FOUND, ...)` si no existe).

1. Añádelo a la interfaz `CatalogoConsultaService`.
2. Impleméntalo en `CatalogoConsultaServiceImpl`, reutilizando `videojuegoRepository`.
3. Úsalo desde `ReviewService` para enriquecer, por ejemplo, `ReviewResumenDTO` con el título del videojuego (tendrás que añadir el campo al DTO).

Sigue exactamente el mismo patrón que ya existía para `existeVideojuego` — la estructura ya la conoces.

Reinicia tu aplicación y comprueba el campo nuevo:

```bash
curl http://localhost:8080/api/v1/videojuegos/1/reviews/resumen
```

**Comprueba**: que la respuesta incluye ahora `tituloVideojuego`, con el título correcto, junto al `totalReviews` y `puntuacionMedia` de siempre.

**Captura**: la respuesta de `/resumen` mostrando el campo `tituloVideojuego`.

---

## Paso 5 — Prueba del componente aislado

Siguiendo el patrón de `@Mock`/`@InjectMocks` de la teoría de este apartado, crea el test en `src/test/java/com/tunombre/gamevault/catalogo/CatalogoConsultaServiceImplTest.java`, con el mismo `package com.tunombre.gamevault.catalogo;` que la implementación. Esto es necesario porque `CatalogoConsultaServiceImpl` es *package-private*: desde un paquete distinto el propio test no podría ni nombrar esa clase.

```java
@ExtendWith(MockitoExtension.class)
class CatalogoConsultaServiceImplTest {

    @Mock
    private VideojuegoRepository videojuegoRepository;

    @InjectMocks
    private CatalogoConsultaServiceImpl catalogoConsultaService;

    @Test
    void existeVideojuego_DebeDevolverTrue_CuandoExiste() {
        when(videojuegoRepository.existsById(1L)).thenReturn(true);

        boolean existe = catalogoConsultaService.existeVideojuego(1L);

        assertTrue(existe);
    }

    @Test
    void existeVideojuego_DebeDevolverFalse_CuandoNoExiste() {
        when(videojuegoRepository.existsById(99L)).thenReturn(false);

        assertFalse(catalogoConsultaService.existeVideojuego(99L));
    }
}
```

`@Mock` sustituye `VideojuegoRepository` por un doble de prueba; `@InjectMocks` construye `CatalogoConsultaServiceImpl` inyectándole ese mock automáticamente — sin arrancar Spring, sin base de datos, un test rapidísimo y totalmente aislado. Añade dos tests más que cubran tu método nuevo `tituloDe`, siguiendo el mismo patrón que ya tienes arriba para `existeVideojuego` — un caso por cada rama: uno donde el videojuego existe (el mock devuelve un `Videojuego` con título), y otro donde no existe (el mock devuelve vacío y compruebas, con `assertThrows`, que se lanza la excepción `404`).

**Captura**: los cuatro tests en verde.

---

## Pregunta final

¿Qué información **no** deberías exponer nunca a través de este componente — por ejemplo, la entidad `Videojuego` completa, con todos sus campos y relaciones? ¿Dónde está la línea entre "exponer lo mínimo necesario para que `reviews` haga su trabajo" y "exponer todo por comodidad, para no tener que pensar en cada método nuevo"? Da un ejemplo concreto de un dato del catálogo que **no** deberías dejar ver a través de `CatalogoConsultaService`.

---

## ✅ Cierre

Ya sabes construir un componente con contrato separado de implementación, dependencias mínimas y explícitas. En la próxima actividad aplicas exactamente el mismo patrón sobre el módulo documental (MongoDB).
