<a id="componentes-bd-objeto-doc"></a>

# 🧩 2. Componentes con BD objeto-relacional y documental

!!! warning "🚧 Contenido pendiente de desarrollo"
    Esta página todavía no tiene la teoría redactada. Usa el prompt de más abajo con
    `/improve-notes`, apoyándote en el proyecto **GameVault** adjunto, para generar el
    contenido definitivo.

---

## Prompt para `/improve-notes`

```text
Redacta el apartado de teoría "Componentes con BD objeto-relacional y documental" del
Tema 5 (RA6 - Componentes de acceso a datos) del módulo Acceso a Datos (0486), semana
real 18 del calendario. Es un apartado de REPASO INTEGRADOR: no introduce tecnología
nueva, sino que revisita JSONB (Tema 3) y MongoDB (Tema 4) desde la óptica de
"componente" ya presentada en el apartado anterior. Sigue las convenciones de estilo del
README.md del repo.

Criterios de evaluación de RA6 que cubre este apartado (curriculum.md):
- f) Componentes que gestionan información almacenada en bases de datos objeto
  relacionales y orientadas a objetos.
- g) Componentes que gestionan información almacenada en una base de datos documental
  nativa.

Apóyate en el proyecto GameVault (com.aleroig.gamevault) como ejemplo real,
reencuadrando piezas ya vistas en temas anteriores bajo la óptica de "componente":
- VideojuegoRepository + VideojuegoSpecifications (Tema 3, JSONB/`jsonb_exists`): ahora
  preséntalo explícitamente como "un componente que encapsula el acceso a una base de
  datos objeto-relacional", cuyo consumidor (VideojuegoService, o cualquier otro
  service) no necesita saber que por debajo hay una función nativa de PostgreSQL como
  `jsonb_exists` — la complejidad queda dentro del componente.
- ReviewRepository + ReviewService (Tema 4, MongoDB): igual, ahora bajo la óptica de
  "componente que gestiona información en una base de datos documental nativa" —
  ReviewService.getResumenByVideojuegoId es un buen ejemplo de lógica que vive DENTRO del
  componente (agregación en memoria) y que el controlador no necesita conocer.
- Retoma el componente CatalogoConsultaService (construido en la Actividad 4.1 y
  analizado/extendido en la Actividad 5.1): explica cómo ese mismo patrón (interfaz +
  implementación oculta) puede aplicarse igual de bien para exponer, desde `reviews`
  hacia otros módulos, un `ReviewsConsultaService` análogo (por ejemplo, "cuántas
  reseñas tiene un videojuego") — que es exactamente lo que se construirá en la
  Actividad 5.2 — cerrando el círculo de que el mismo patrón de componente sirve para
  JPA, JSONB y MongoDB por igual: la interfaz no necesita saber qué motor hay debajo.

Cierra con una idea explícita de síntesis: da igual la tecnología de persistencia
(relacional, objeto-relacional con JSONB, documental con MongoDB) — el patrón de
"componente con interfaz clara + implementación oculta" es el mismo en los tres casos, y
eso es precisamente lo que se pide demostrar en RA6.

No entres en pruebas ni en integración final: eso es el siguiente apartado,
integracion-y-pruebas.md, que cierra el RA6.
```
