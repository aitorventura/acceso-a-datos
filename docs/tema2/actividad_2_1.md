# 🧪 Actividad 2.1: De JDBC puro a Hibernate — mapeo con anotaciones

!!! warning "🚧 Contenido pendiente de desarrollo"
    Esta actividad todavía no está redactada. Usa el prompt de más abajo con
    `/improve-notes`, apoyándote en el proyecto **GameVault** adjunto, para generar el
    enunciado definitivo.

---

## Prompt para `/improve-notes`

```text
Redacta la Actividad 2.1 del Tema 2 (RA3 - ORM) del módulo Acceso a Datos (0486), semana
real 7 del calendario. Sigue el patrón de estructura de docs/tema0/actividad_0_6.md y
usa la skill /actividad-plantilla-acceso-a-datos si necesita plantilla/solución en .docx.

IMPORTANTE — enfoque: es una PRÁCTICA GUIADA, no un reto. El alumnado trabaja sobre su
propia copia de GameVault (el mismo proyecto adjunto, construido individualmente durante
el curso). El enunciado debe guiar paso a paso, mostrando el código y explicando cada
decisión; solo se deja sin guiar, como mini-reto, lo que repita un patrón idéntico ya
mostrado en la misma actividad.

Objetivo (RA3, criterios a, b, c): que el alumnado, sobre su GameVault (con
Videojuego/Estudio ya creados desde el Tema 1), revise y comprenda a fondo el mapeo de
sus entidades, guiado con experimentos concretos sobre decisiones reales del proyecto:

- com/aleroig/gamevault/catalogo/Videojuego.java: revisar guiadamente
  `@GeneratedValue(strategy = GenerationType.IDENTITY)` (explicando en el enunciado la
  alternativa `SEQUENCE`/`AUTO` y por qué IDENTITY encaja bien con PostgreSQL).
- `@Column(precision = 10, scale = 2)` sobre el precio: experimento guiado — quitar la
  anotación, arrancar, mirar el tipo de columna generado, volver a ponerla y comparar
  (pasos e instrucciones de verificación dados en el enunciado).
- Experimento guiado con `fetch = FetchType.LAZY` en la relación @ManyToOne hacia
  Estudio: activar el log de SQL (`spring.jpa.show-sql=true`/`org.hibernate.SQL=DEBUG` —
  configuración dada), hacer un `findAll()` con LAZY y con EAGER, pegar el SQL generado
  en cada caso y comentarlo — el enunciado indica exactamente qué mirar en el log.
- Experimento guiado de cierre: probar qué pasa al borrar un Estudio con Videojuego
  asociados, con y sin `cascade = CascadeType.ALL, orphanRemoval = true`, siguiendo los
  pasos dados, y describir la diferencia observada + pregunta de comprensión (¿por qué
  la cascada va en ese lado de la relación y no al revés?).

Esta actividad no debe usar Specifications ni @Query JPQL: eso corresponde a las
actividades 2.2 y 2.3. El foco es exclusivamente el mapeo con anotaciones y el ciclo de
vida de las entidades.
```
