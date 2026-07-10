<a id="instalacion-configuracion-orm"></a>

# 🧩 1. Instalación y configuración de Hibernate

!!! warning "🚧 Contenido pendiente de desarrollo"
    Esta página todavía no tiene la teoría redactada. Usa el prompt de más abajo con
    `/improve-notes`, apoyándote en el proyecto **GameVault** adjunto, para generar el
    contenido definitivo.

---

## Prompt para `/improve-notes`

```text
Redacta el apartado de teoría "Instalación y configuración de Hibernate" del Tema 2 (RA3
- ORM) del módulo Acceso a Datos (0486), semana real 7 del calendario. Sigue las
convenciones de estilo del README.md del repo (tabs-colored, admonitions, densidad
visual, pretérito perfecto compuesto).

Criterios de evaluación de RA3 que cubre este apartado (curriculum.md):
- a) Se ha instalado la herramienta ORM.
- b) Se ha configurado la herramienta ORM.
- c) Se han definido configuraciones de mapeo.

ESTRUCTURA OBLIGATORIA — teoría primero, proyecto después.

PARTE 1 — Teoría general, desde cero y sin mencionar GameVault todavía:
- Qué es el mapeo objeto-relacional como idea: retoma el desfase objeto-relacional del
  Tema 1 y presenta la solución — una herramienta que traduce automáticamente objetos ↔
  filas siguiendo unas reglas de mapeo que tú declaras una vez. Ilustra con el ejemplo
  genérico mínimo (clase Alumno ↔ tabla alumno): qué escribías a mano con JDBC en el
  Tema 1 y qué desaparece con un ORM.
- El trío de nombres que siempre se confunde, aclarado de una vez: ORM (el concepto),
  JPA (la especificación estándar de Java: las anotaciones e interfaces), Hibernate (la
  implementación más usada de JPA). Añade dónde encaja Spring Data JPA (una capa más de
  comodidad por encima: los repositorios). Un diagrama de capas ayuda.
- Qué es una clase persistente/entidad y las dos formas históricas de declarar el mapeo
  (fichero XML de mapeo vs. anotaciones — el currículo pide conocer ambas; hoy se usan
  anotaciones y el XML se menciona como legado).
- Los estados de un objeto en el ORM (transient, managed/persistent, detached, removed)
  explicados con un objeto genérico y un diagrama de estados, ANTES de verlos sobre
  Videojuego.

PARTE 2 — Aterrizaje en GameVault: explica que "instalar" Hibernate en un proyecto
Spring Boot no es un paso manual, sino una dependencia (`spring-boot-starter-data-jpa`
en el `pom.xml`) que ya incluye Hibernate como implementación por defecto — apóyate en
el pom.xml real de GameVault para mostrar esa dependencia. Después:
- com/aleroig/gamevault/catalogo/Videojuego.java y Estudio.java: mapeo basado en
  anotaciones (`@Entity`, `@Table(name = "...")`, `@Id`, `@GeneratedValue(strategy =
  GenerationType.IDENTITY)`, `@Column(precision = 10, scale = 2)` en el precio,
  `@ManyToOne(fetch = FetchType.LAZY)` + `@JoinColumn`, `@OneToMany(mappedBy = ...,
  cascade = CascadeType.ALL, orphanRemoval = true)`) — explica cada anotación una a una,
  relacionándola con lo que el alumnado ya vio "a pelo" en el Tema 1 con JDBC puro.
- com/aleroig/gamevault/config/PersistenceConfig.java y
  com/aleroig/gamevault/config/Jackson3HibernateMapper.java: configuración específica de
  Hibernate del proyecto — coméntalos como ejemplo de configuración avanzada (sin entrar
  en el detalle exacto de FormatMapper si complica demasiado la explicación a este nivel;
  puedes simplificarlo a "Hibernate necesita a veces ayuda extra para mapear tipos como
  JSON, y aquí se configura esa ayuda").
- La propiedad `spring.jpa.hibernate.ddl-auto` en application-dev.yaml, ya introducida en
  el Tema 1, retómala aquí desde el ángulo de "esto es configuración del ORM, no del
  conector" — distingue claramente configuración de conexión (Tema 1) de configuración de
  mapeo/ORM (este tema).

Cierra retomando los estados del objeto de la PARTE 1, ahora con Videojuego como
ejemplo concreto en cada estado. No entres todavía en `save()`/`findById()` ni en
Specifications: eso es el apartado persistencia-objetos.md, siguiente en este mismo
tema.
```
