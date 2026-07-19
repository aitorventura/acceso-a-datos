# 🧪 Actividad 4.1: `CatalogoConsultaService` — un componente reutilizable

!!! info "Práctica guiada"
    Relees el componente que ya construiste en el Tema 3, esta vez bajo la óptica de "componente" de la teoría, y lo extiendes con un método nuevo siguiendo el mismo patrón.

## Qué vas a practicar

- Analizar críticamente por qué un componente ya construido está bien diseñado.
- Razonar sobre las consecuencias del desacoplamiento con un caso concreto.
- Extender un componente existente siguiendo su propio patrón.

---

## Requisitos previos

Tu `CatalogoConsultaService`/`CatalogoConsultaServiceImpl` de la Actividad 3.1.

---

## Paso 1 — Relectura guiada por preguntas

Abre tu propio código y responde, con el código delante:

1. ¿Por qué `CatalogoConsultaService` (la interfaz) vive en el paquete `catalogo.api`, y `CatalogoConsultaServiceImpl` no?
2. ¿Por qué `CatalogoConsultaServiceImpl` no lleva el modificador `public`? ¿Qué pasaría si lo añadieras — cambiaría algo en cómo funciona, o solo en qué es visible desde fuera del paquete?
3. En `ReviewService`, ¿qué tipo declaras al inyectar esta dependencia — la interfaz `CatalogoConsultaService`, o la clase `CatalogoConsultaServiceImpl`? ¿Podrías, siquiera, declarar el tipo concreto desde otro paquete?

---

## Paso 2 — El desacoplamiento, con un caso concreto

Imagina que mañana decides sustituir JPA por otra tecnología de persistencia para el catálogo (no lo vas a hacer, es un ejercicio de razonamiento). **Responde**:

- ¿Qué ficheros del paquete `catalogo` tendrías que tocar?
- ¿Tendrías que tocar algo de `reviews`?
- ¿Qué papel concreto juega la interfaz `CatalogoConsultaService` en que la respuesta a la pregunta anterior sea "no"?

---

## Mini-reto — un segundo método

Sin más código dado que el patrón que ya tienes delante, añade un segundo método al componente: `String tituloDe(Long videojuegoId)`, que devuelva el título de un videojuego (o lance `ResponseStatusException(HttpStatus.NOT_FOUND, ...)` si no existe).

1. Añádelo a la interfaz `CatalogoConsultaService`.
2. Impleméntalo en `CatalogoConsultaServiceImpl`, reutilizando `videojuegoRepository`.
3. Úsalo desde `ReviewService` para enriquecer, por ejemplo, `ReviewResumenDTO` con el título del videojuego (tendrás que añadir el campo al DTO).

Sigue exactamente el mismo patrón que ya existía para `existeVideojuego` — la estructura ya la conoces.

---

## Paso 3 — Prueba del componente aislado

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

`@Mock` sustituye `VideojuegoRepository` por un doble de prueba; `@InjectMocks` construye `CatalogoConsultaServiceImpl` inyectándole ese mock automáticamente — sin arrancar Spring, sin base de datos, un test rapidísimo y totalmente aislado. Añade un tercer test que cubra tu método nuevo `tituloDe`, siguiendo el mismo patrón.

---

## Pregunta final

¿Qué información **no** deberías exponer nunca a través de este componente — por ejemplo, la entidad `Videojuego` completa, con todos sus campos y relaciones? ¿Dónde está la línea entre "exponer lo mínimo necesario para que `reviews` haga su trabajo" y "exponer todo por comodidad, para no tener que pensar en cada método nuevo"? Da un ejemplo concreto de un dato del catálogo que **no** deberías dejar ver a través de `CatalogoConsultaService`.

---

## ✅ Cierre

Ya sabes leer tu propio código con ojo crítico de "componente" — contrato separado de implementación, dependencias mínimas y explícitas. En la próxima actividad aplicas exactamente el mismo patrón sobre el módulo documental (MongoDB).
