<a id="componentes-conectores-orm"></a>

# 🧩 1. Componentes con conectores y ORM

Último tema del módulo. Todo lo que has construido hasta ahora — repositorios, servicios, controladores — lo vas a mirar ahora bajo una óptica distinta: la de **componente**. No vas a escribir código radicalmente nuevo; vas a entender por qué el código que ya tienes está bien diseñado (o cómo mejorarlo) desde este ángulo.

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

!!! info "Sobre componentes con ficheros"
    El currículo también menciona "componentes que gestionan información almacenada en ficheros" — ese contenido se trabaja en la FCT, en la empresa, no en este módulo, ya que corresponde a la gestión de ficheros. Lo mencionamos aquí solo para que sepas que existe, sin profundizar.

### Nivel 1: un repositorio, ya es un componente

```java
public interface LibroRepository extends JpaRepository<Libro, Long> {
}
```

En su forma más básica, `LibroRepository` ya es un "componente con conector a base de datos": una interfaz que Spring implementa y gestiona por ti — se declara una vez (`private final LibroRepository libroRepository;`) y se reutiliza donde haga falta, sin que nadie que lo use necesite saber cómo genera Spring esa implementación por debajo.

### Nivel 2: `CatalogoConsultaService`, el ejemplo mejor diseñado

```java
// El contrato — en el paquete api, pensado para que otros módulos lo vean
public interface CatalogoConsultaService {
    boolean existeLibro(Long libroId);
}

// La implementación — oculta, sin el modificador public
@Service
class CatalogoConsultaServiceImpl implements CatalogoConsultaService {
    private final LibroRepository libroRepository;

    @Override
    public boolean existeLibro(Long libroId) {
        return libroRepository.existsById(libroId);
    }
}
```

¿Por qué es tan buen ejemplo? Porque separa con precisión el **contrato** (la interfaz, en un paquete `api`, visible para quien la necesite) de la **implementación concreta** (la clase, sin modificador `public`, invisible fuera de su propio paquete). El módulo de reseñas del Tema 3 usa `CatalogoConsultaService.existeLibro(...)` sin conocer ni el paquete `catalogo` internamente, ni cómo está implementado ese método — desacoplamiento real entre módulos. Es, formalizado, el mismo patrón de integridad referencial manual que ya usaste allí.

### El contraste que ilustra el desacoplamiento

Técnicamente, el servicio de reseñas **podría** inyectar `LibroRepository` directamente y llamar a `existsById(...)` él mismo, ahorrándose la interfaz intermedia. ¿Por qué no se hace así? Porque eso rompería el aislamiento entre módulos: `resenas` pasaría a depender de un detalle interno de `catalogo` (su repositorio JPA concreto), y cualquier cambio en cómo `catalogo` gestiona su persistencia (cambiar de JPA a otra cosa, por ejemplo) obligaría a tocar también `resenas`. Con el componente de por medio, ese cambio quedaría contenido dentro de `catalogo` — `resenas` seguiría llamando a la misma interfaz, sin enterarse de nada. Esta es la ventaja de sustituibilidad de la tabla de arriba, hecha concreta.

---

## ✅ Ideas clave

??? tip "Abrir resumen"

    - Un **componente** es una pieza autocontenida y reutilizable, con una responsabilidad y un contrato público que oculta su implementación.
    - El origen histórico son los JavaBeans; en Spring Boot, el equivalente son los **beans gestionados por inyección de dependencias**.
    - Ventajas: reutilización, sustituibilidad, pruebas aisladas, división del trabajo. Inconveniente: más interfaces e indirección.
    - Spring (`@Service`/`@Repository`/`@Component` + inyección) ES la "herramienta de desarrollo de componentes" en este contexto — no hay paleta visual.
    - Un `JpaRepository` ya es un componente básico; `CatalogoConsultaService`/`CatalogoConsultaServiceImpl` es el ejemplo mejor diseñado: contrato en `api`, implementación oculta, desacoplamiento real entre módulos.
