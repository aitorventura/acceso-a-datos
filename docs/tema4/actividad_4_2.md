# 🧪 Actividad 4.2: Mismo patrón, otro motor — un componente sobre el módulo documental

!!! warning "Descarga la plantilla"
    📄 [Plantilla 4.2 — Mismo patrón, otro motor: un componente sobre el módulo documental](plantillas/Actividad_4_2_AD_Plantilla.docx){target="_blank" rel="noopener"}

!!! info "Práctica guiada — paso de mayor peso"
    Hoy repites, sobre MongoDB, el mismo patrón exacto de componente que ya has construido sobre PostgreSQL en la Actividad 4.1: casi todo es aplicar, sobre un motor distinto, una estructura que ya conoces.

## Qué vas a practicar

- Replicar el patrón interfaz + implementación oculta sobre un motor distinto.
- Usar un componente nuevo desde otro módulo, sin conocer su implementación.
- Comparar dos componentes que resuelven el mismo problema sobre motores distintos.

---

## Requisitos previos

Tu `CatalogoConsultaService` (Actividad 4.1) como referencia visual constante.

---

## Paso 1 — Planteamiento: el contrato mínimo

¿Qué necesitaría saber el módulo `catalogo` (u otro futuro) sobre las reseñas, sin conocer nada de MongoDB? Define tú mismo el contrato mínimo: dos métodos, uno que devuelva cuántas reseñas tiene un videojuego (`long`) y otro que devuelva su puntuación media (`double`), los dos recibiendo el `id` del videojuego. La lógica de ambos **ya existe** dentro de `ReviewService.getResumenByVideojuegoId` — no la vas a reescribir, solo a exponerla como componente.

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

Enriquece `VideojuegoResponseDTO` con la puntuación media, usando el componente nuevo desde `catalogo`:

```java
public record VideojuegoResponseDTO(
        Long id,
        String titulo,
        BigDecimal precio,
        LocalDate fechaLanzamiento,
        String nombreEstudio,
        Map<String, Object> detallesPlataforma,
        Double puntuacionMedia // campo nuevo
) {}
```

```java
// En VideojuegoService, inyecta ReviewsConsultaService (la interfaz, no la implementación)
private VideojuegoResponseDTO mapToDTO(Videojuego v) {
    Double media = reviewsConsultaService.puntuacionMediaDe(v.getId());
    return new VideojuegoResponseDTO(
            v.getId(), v.getTitulo(), v.getPrecio(), v.getFechaLanzamiento(),
            v.getEstudio().getNombre(), v.getDetallesPlataforma(), media
    );
}
```

**Fíjate**: `catalogo` no importa nada de `reviews` salvo la interfaz del paquete `api` — ni conoce `Review`, ni `ReviewRepository`, ni cómo se calcula la media por dentro.

---

## Paso 4 — Verificación

```bash
curl http://localhost:8080/api/v1/videojuegos/1
```

**Comprueba**: que la respuesta incluye `puntuacionMedia`, y que el dato coincide con lo que calcula `GET /api/v1/videojuegos/{id}/reviews/resumen` para el mismo videojuego — están usando la misma lógica por debajo, expuesta ahora desde dos sitios distintos.

**Captura**: la respuesta de `GET /videojuegos/1` con `puntuacionMedia`, junto a la de `GET /reviews/resumen` mostrando el mismo valor.

**Ejecuta también** tus tests existentes (`ReviewServiceTest`, `VideojuegoServiceTest`, etc.) y comprueba que siguen pasando.

**Captura**: tu batería de tests completa en verde.

---

## Paso 5 — Prueba del componente aislado

Con `ReviewsConsultaService` ya construido, integrado en `catalogo` y verificado de extremo a extremo, toca aislarlo: un test aislado, siguiendo exactamente el patrón de `CatalogoConsultaServiceImplTest` (mock del repositorio, `@InjectMocks` sobre la implementación).

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

**Una pregunta más**: con `CatalogoConsultaService` (Actividad 4.1) y `ReviewsConsultaService` (hoy), `catalogo` y `reviews` pasan a depender el uno del otro — cada módulo expone un contrato que el otro consume. ¿Te parece un problema que dos módulos dependan mutuamente el uno del otro? Razónalo (pista: fíjate en qué depende de qué exactamente — ¿es `VideojuegoService` quien depende de `ReviewService`, o de algo más concreto?).

---

## ✅ Cierre

Tienes dos componentes, sobre dos motores completamente distintos, con el mismo patrón de diseño exacto. En la última actividad del módulo integras todo lo construido en un test final de extremo a extremo.
