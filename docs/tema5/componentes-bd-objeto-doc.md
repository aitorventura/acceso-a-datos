<a id="componentes-bd-objeto-doc"></a>

# 🧩 2. Componentes con BD objeto-relacional y documental

Este apartado no introduce ninguna tecnología nueva — es un repaso integrador. Revisitas JSONB (Tema 3) y MongoDB (Tema 4), esta vez con la óptica de "componente" que acabas de conocer en el apartado anterior.

---

## 🐘 JSONB, visto como componente

```java
public interface VideojuegoRepository extends JpaRepository<Videojuego, Long>, JpaSpecificationExecutor<Videojuego> {
}
```

`VideojuegoRepository`, junto con `VideojuegoSpecifications` (donde vive `disponibleEnPlataforma`, con su `jsonb_exists`), es un **componente que encapsula el acceso a una base de datos objeto-relacional**. Quien lo consume — `VideojuegoService`, o cualquier otro service que pudiera necesitarlo — no tiene ni que saber que, por debajo de una de sus Specifications, hay una función nativa de PostgreSQL como `jsonb_exists`. Esa complejidad queda completamente contenida dentro del componente; fuera de él, es "una Specification más".

---

## 🍃 MongoDB, visto como componente

```java
public ReviewResumenDTO getResumenByVideojuegoId(Long videojuegoId) {
    List<Review> reviews = reviewRepository.findByVideojuegoId(videojuegoId);
    long totalReviews = reviews.size();
    double puntuacionMedia = reviews.stream().mapToInt(Review::getPuntuacion).average().orElse(0.0);
    return new ReviewResumenDTO(videojuegoId, totalReviews, puntuacionMedia);
}
```

`ReviewRepository` + `ReviewService` es, bajo esta misma óptica, un "componente que gestiona información almacenada en una base de datos documental nativa". `getResumenByVideojuegoId` es un buen ejemplo de lógica que vive **dentro** del componente — la agregación en memoria (contar, calcular la media) — sin que el controlador que lo llama necesite saber cómo se ha calculado ese resumen, ni que por debajo hay documentos de MongoDB en vez de filas de una tabla.

---

## 🪞 Cerrando el círculo: el mismo patrón, tres motores distintos

Ya construiste `CatalogoConsultaService` en el Tema 4 (y lo extendiste en la Actividad 5.1) — interfaz en `catalogo.api`, implementación oculta. Ese mismo patrón se puede aplicar, exactamente igual, para exponer desde `reviews` hacia otros módulos un componente análogo — por ejemplo, `ReviewsConsultaService`, con algo como "cuántas reseñas tiene un videojuego". Es justo lo que vas a construir en la Actividad 5.2.

```mermaid
flowchart TB
    subgraph JPA["🐘 PostgreSQL puro"]
        A["VideojuegoRepository"]
    end
    subgraph JSONB["🐘 PostgreSQL + JSONB"]
        B["VideojuegoSpecifications<br/>disponibleEnPlataforma"]
    end
    subgraph MONGO["🍃 MongoDB"]
        C["ReviewRepository +<br/>ReviewService"]
    end
    A --> D["🧩 Mismo patrón de componente:<br/>interfaz + implementación oculta"]
    B --> D
    C --> D
```

La idea de síntesis de este apartado: **da igual la tecnología de persistencia** — relacional puro, objeto-relacional con JSONB, documental con MongoDB — el patrón de "componente con interfaz clara + implementación oculta" es el mismo en los tres casos. La interfaz nunca necesita saber qué motor hay por debajo. Eso, precisamente, es el objetivo de este tema: no una tecnología concreta, sino una forma de diseñar que funciona igual de bien sobre cualquiera de ellas.

---

## ✅ Ideas clave

??? tip "Abrir resumen"

    - `VideojuegoRepository`/`VideojuegoSpecifications` es un componente que encapsula acceso objeto-relacional (JSONB) — su complejidad interna (`jsonb_exists`) queda oculta a quien lo usa.
    - `ReviewRepository`/`ReviewService` es un componente que encapsula acceso documental (MongoDB) — su lógica de agregación vive dentro, no en el controlador.
    - El mismo patrón (interfaz + implementación oculta) de `CatalogoConsultaService` se puede replicar sobre cualquier motor — es lo que vas a hacer con `ReviewsConsultaService`.
    - La tecnología de persistencia por debajo es irrelevante para el patrón de componente — esa independencia es exactamente la idea clave de este tema.
