<a id="componentes-conectores-orm"></a>

# 🧩 1. Componentes con conectores y ORM

Último tema del módulo. Todo lo que has construido hasta ahora — repositorios, servicios, controladores — lo vas a mirar ahora bajo una óptica distinta: la de **componente**. Vas a identificar un problema que ya tienes en tu propio código desde el Tema 3, y resolverlo construyendo tu primer componente con un contrato explícito.

---

## 🧩 Qué es la programación orientada a componentes

La **programación orientada a componentes** consiste en construir aplicaciones ensamblando piezas autocontenidas y reutilizables — cada una con una responsabilidad clara y un **contrato público** (su interfaz), que oculta cómo resuelve esa responsabilidad por dentro. Quien usa un componente solo necesita conocer su contrato, nunca su implementación.

El currículo asocia a un componente varias características clásicas:

- **Propiedades**: valores configurables del componente.
- **Eventos**: acciones que el componente puede disparar, a las que otros pueden reaccionar.
- **Persistencia/serialización**: la capacidad de guardar y recuperar el estado del componente.
- **Empaquetado**: la forma de distribuirlo para que otros lo usen sin ver su código fuente.

### El origen histórico: JavaBeans

Estas ideas vienen de los **JavaBeans** y los componentes visuales que, en entornos de desarrollo antiguos, se arrastraban desde una paleta del IDE — un botón, una rejilla de datos — y se configuraban ajustando sus propiedades desde un panel, sin escribir código. De ahí sale el vocabulario del currículo: propiedades, eventos, empaquetado.

### La traducción al mundo actual

En un proyecto Spring Boot moderno, el equivalente conceptual de aquellos JavaBeans son los **Spring Beans gestionados por inyección de dependencias** — ya los conoces desde el Tema 1. Una clase con una responsabilidad clara, con su interfaz, que se declara una vez y se reutiliza donde haga falta, sin que quien la usa conozca los detalles internos. La idea de fondo es exactamente la misma: piezas autocontenidas, con un contrato, ensambladas por un contenedor (el IDE antes, Spring ahora).

### Ventajas e inconvenientes

| Ventajas | Inconvenientes |
|---|---|
| Reutilización — la misma pieza sirve en varios sitios | Más interfaces que mantener |
| Sustituibilidad — cambias la implementación sin tocar quien la usa | Más indirección — seguir el flujo de código cuesta un poco más |
| Pruebas aisladas — pruebas cada pieza por separado | |
| División del trabajo — cada componente, una responsabilidad | |

---

## 📚 La progresión de "componente", con la librería

### Herramientas de desarrollo de componentes

Aquí no aplica una herramienta visual de arrastrar y soltar — sería un anacronismo en un proyecto backend. La "herramienta" en este contexto es **Spring** en sí mismo: vía `@Service`, `@Repository`, `@Component` y la inyección con `@RequiredArgsConstructor`, es Spring quien gestiona el ciclo de vida y el ensamblado de los componentes de tu aplicación.

### Nivel 1: un repositorio, ya es un componente

```java
public interface LibroRepository extends JpaRepository<Libro, Long> {
}
```

En su forma más básica, `LibroRepository` ya es un "componente con conector a base de datos": una interfaz que Spring implementa y gestiona por ti — se declara una vez (`private final LibroRepository libroRepository;`) y se reutiliza donde haga falta, sin que nadie que lo use necesite saber cómo genera Spring esa implementación por debajo.

### Nivel 2: cuando un repositorio no basta

Recuerda `NotaLecturaService`, del Tema 3: para comprobar que un libro existe antes de tocar Mongo, inyecta `LibroRepository` directamente:

```java
// NotaLecturaService, tal y como lo dejaste en el Tema 3
private final LibroRepository libroRepository;

public List<NotaLecturaResponseDTO> findByLibroId(Long libroId) {
    if (!libroRepository.existsById(libroId)) {
        throw new ResponseStatusException(HttpStatus.NOT_FOUND, "Libro no encontrado en el catálogo");
    }
    // ...
}
```

Funciona, pero acopla el módulo de notas de lectura a un detalle muy interno del módulo del catálogo: su repositorio JPA concreto. Si el catálogo cambiara mañana cómo organiza internamente su persistencia, tendrías que tocar también `NotaLecturaService`, aunque no tenga nada que ver con ese cambio. Este es exactamente el problema que resuelve un componente bien diseñado: exponer solo un contrato mínimo, y esconder todo lo demás detrás.

### Construyendo el componente: interfaz + implementación oculta

La solución es una interfaz pequeña, pensada para que otros módulos la usen, con una implementación que sí conoce los detalles pero permanece oculta. El contrato, en un paquete `api` — pensado para lo que otros módulos SÍ pueden ver:

```java
public interface LibroConsultaService {
    boolean existeLibro(Long libroId);
}
```

Y su implementación, en el paquete del catálogo, no en `api` — fíjate en que la clase no lleva `public`: es *package-private*, invisible fuera de su propio paquete, así que nadie puede depender de ella por accidente:

```java
@Service
@RequiredArgsConstructor
class LibroConsultaServiceImpl implements LibroConsultaService {

    private final LibroRepository libroRepository;

    @Override
    public boolean existeLibro(Long libroId) {
        return libroRepository.existsById(libroId);
    }
}
```

`NotaLecturaService` pasa a depender solo de la interfaz, nunca de `LibroRepository` ni de la clase que la implementa:

```java
// NotaLecturaService, ahora
private final LibroConsultaService libroConsultaService;

public List<NotaLecturaResponseDTO> findByLibroId(Long libroId) {
    if (!libroConsultaService.existeLibro(libroId)) {
        throw new ResponseStatusException(HttpStatus.NOT_FOUND, "Libro no encontrado en el catálogo");
    }
    // ...
}
```

### El contraste que ilustra el desacoplamiento

Técnicamente, el módulo de notas de lectura **podría** seguir inyectando `LibroRepository` directamente, ahorrándose la interfaz intermedia. ¿Por qué merece la pena el paso extra? Porque con el componente de por medio, cualquier cambio en cómo el catálogo gestiona su persistencia (cambiar de JPA a otra cosa, por ejemplo) queda contenido dentro de su propio módulo — el módulo de notas de lectura seguiría llamando a la misma interfaz, sin enterarse de nada. Esta es la ventaja de sustituibilidad de la tabla de arriba, hecha concreta.

Vas a construir exactamente este mismo patrón sobre tu propio proyecto en la Actividad 4.1, con `Videojuego`/`Review` en lugar de `Libro`/`NotaLectura`.

---

## ✅ Ideas clave

??? tip "Abrir resumen"

    - Un **componente** es una pieza autocontenida y reutilizable, con una responsabilidad y un contrato público que oculta su implementación.
    - El origen histórico son los JavaBeans; en Spring Boot, el equivalente son los **beans gestionados por inyección de dependencias**.
    - Ventajas: reutilización, sustituibilidad, pruebas aisladas, división del trabajo. Inconveniente: más interfaces e indirección.
    - Spring (`@Service`/`@Repository`/`@Component` + inyección) ES la "herramienta de desarrollo de componentes" en este contexto — no hay paleta visual.
    - Un `JpaRepository` ya es un componente básico; una interfaz + implementación oculta (`LibroConsultaService`/`LibroConsultaServiceImpl`) es el siguiente nivel: contrato en `api`, implementación oculta, desacoplamiento real entre módulos.
