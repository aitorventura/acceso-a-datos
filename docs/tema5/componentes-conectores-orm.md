<a id="componentes-conectores-orm"></a>

# 🧩 1. Componentes con conectores y ORM

!!! warning "🚧 Contenido pendiente de desarrollo"
    Esta página todavía no tiene la teoría redactada. Usa el prompt de más abajo con
    `/improve-notes`, apoyándote en el proyecto **GameVault** adjunto, para generar el
    contenido definitivo.

---

## Prompt para `/improve-notes`

```text
Redacta el apartado de teoría "Componentes con conectores y ORM" del Tema 5 (RA6 -
Componentes de acceso a datos) del módulo Acceso a Datos (0486), semana real 17 del
calendario. Sigue las convenciones de estilo del README.md del repo.

Criterios de evaluación de RA6 que cubre este apartado (curriculum.md):
- a) Ventajas e inconvenientes de utilizar programación orientada a componentes.
- b) Herramientas de desarrollo de componentes.
- c) Componentes que gestionan información almacenada en ficheros — trátalo solo de
  pasada: el calendario marca este criterio como "parcial, RA1 se trabaja en la FCT
  (empresa)", así que no profundices, solo menciona que existe y por qué no se cubre aquí
  en detalle.
- d) Componentes con conectores a bases de datos.
- e) Componentes con ORM.

ESTRUCTURA OBLIGATORIA — teoría primero, proyecto después.

PARTE 1 — Teoría general, desde cero:
- Qué es la programación orientada a componentes: construir aplicaciones ensamblando
  piezas autocontenidas y reutilizables, cada una con una responsabilidad y un contrato
  público (su interfaz), ocultando el interior. Características que el currículo asocia
  a un componente: propiedades, eventos, persistencia/serialización, empaquetado — cada
  una definida en 1-2 frases.
- El origen histórico que el currículo tiene en mente: los JavaBeans y los componentes
  visuales que se arrastraban en una paleta del IDE (un botón, una rejilla de datos) y
  se configuraban por propiedades — cuéntalo brevemente para que las palabras del
  currículo tengan sentido.
- La traducción al mundo actual: en un proyecto Spring Boot moderno el equivalente
  conceptual son los **Spring Beans gestionados por inyección de dependencias**: una
  clase con una responsabilidad clara, con su interfaz, que se declara una vez y se
  reutiliza donde haga falta sin que quien la usa conozca los detalles internos.
- Ventajas e inconvenientes de la POC (criterio a) en general: reutilización,
  sustituibilidad, pruebas aisladas, división del trabajo — frente al coste de más
  interfaces y más indirección.

PARTE 2 — Aterrizaje en GameVault. Sobre "herramientas de desarrollo de componentes"
(criterio b): explica que en este contexto no aplica una herramienta visual de arrastrar
y soltar, sino que Spring (vía `@Service`, `@Repository`, `@Component` e inyección con
`@RequiredArgsConstructor`) ES la "herramienta" que gestiona el ciclo de vida y el
ensamblado de componentes.

Apóyate en el proyecto GameVault (com.aleroig.gamevault) como ejemplo real, mostrando la
progresión de "componente" de menor a mayor especialización:
- com/aleroig/gamevault/catalogo/VideojuegoRepository.java y EstudioRepository.java:
  ejemplo de "componente con conector a base de datos" en su forma más básica — una
  interfaz `JpaRepository` es, en sí misma, un componente reutilizable (Spring genera la
  implementación y la inyecta donde se declare `private final VideojuegoRepository ...`).
- com/aleroig/gamevault/catalogo/api/CatalogoConsultaService.java (la interfaz) y
  com/aleroig/gamevault/catalogo/CatalogoConsultaServiceImpl.java (la implementación,
  package-private, anotada `@Service`): este es el ejemplo MÁS CLARO de "componente" bien
  diseñado en GameVault — expón por qué: separa contrato (interfaz, en el paquete `api`)
  de implementación concreta (clase, sin modificador `public`, oculta fuera de su
  paquete), y la usa `ReviewService` (del módulo `reviews`) sin conocer ni el paquete
  `catalogo` interno ni cómo se implementa `existeVideojuego` — desacoplamiento real entre
  módulos, documentado en docs/arquitectura/modulos-y-desacoplamiento.md del proyecto.
- Contrasta esto con inyectar `VideojuegoRepository` directamente desde `reviews`
  (que SÍ sería posible técnicamente, pero rompería el aislamiento entre módulos): usa
  esta comparación para el criterio a) (ventajas e inconvenientes de POC) de forma
  concreta, no abstracta.

No entres todavía en componentes con JSONB/documentales: eso es el siguiente apartado,
componentes-bd-objeto-doc.md.
```
