# 🧪 Actividad 4.2: Repaso integrador — un componente sobre el módulo documental

!!! info "Práctica guiada — mini-reto de mayor peso"
    Hoy repites, sobre MongoDB, el mismo patrón exacto de componente que ya construiste sobre PostgreSQL en la Actividad 4.1. Es una actividad de repaso integrador: casi todo es aplicar una estructura que ya conoces.

## Qué vas a practicar

- Replicar el patrón interfaz + implementación oculta sobre un motor distinto.
- Usar un componente nuevo desde otro módulo, sin conocer su implementación.
- Comparar dos componentes que resuelven el mismo problema sobre motores distintos.

---

## Requisitos previos

Tu `CatalogoConsultaService` (Actividades 3.1 y 4.1) como referencia visual constante.

---

## Paso 1 — Planteamiento: el contrato mínimo

¿Qué necesitaría saber el módulo `catalogo` (u otro futuro) sobre las reseñas, sin conocer nada de MongoDB? Define el contrato:

```java
public interface ReviewsConsultaService {
    long totalReviewsDe(Long videojuegoId);
    double puntuacionMediaDe(Long videojuegoId);
}
```

La lógica de estos dos métodos **ya existe** dentro de `ReviewService.getResumenByVideojuegoId` — no la vas a reescribir, solo a exponerla como componente.

---

## Mini-reto (con más peso de lo habitual) — el componente completo

!!! warning "El paquete `reviews.api` no existe todavía en tu proyecto"
    A diferencia de `catalogo.api` (que ya tienes de la Actividad 3.1), aquí no hay ningún precedente — el primer paso es crear la carpeta nueva.

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
3. Un test aislado, siguiendo exactamente el patrón de `CatalogoConsultaServiceImplTest` (mock del repositorio, `@InjectMocks` sobre la implementación).

---

## Paso 2 — Usarlo desde `catalogo`

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

## Paso 3 — Verificación

```bash
curl http://localhost:8080/api/v1/videojuegos/1
```

**Comprueba**: que la respuesta incluye `puntuacionMedia`, y que el dato coincide con lo que calcula `GET /api/v1/videojuegos/{id}/reviews/resumen` para el mismo videojuego — están usando la misma lógica por debajo, expuesta ahora desde dos sitios distintos. **Ejecuta también** tus tests existentes (`ReviewServiceTest`, `VideojuegoServiceTest`, etc.) y comprueba que siguen pasando.

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

---

## ✅ Cierre

Tienes dos componentes, sobre dos motores completamente distintos, con el mismo patrón de diseño exacto. En la última actividad del módulo integras todo lo construido en un test final de extremo a extremo.
