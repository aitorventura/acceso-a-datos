<a id="persistencia-objetos"></a>

# 🧩 2. Persistencia y recuperación de objetos

Ya sabes cómo se mapea una entidad. Toca ahora usar ese mapeo para lo que de verdad importa: guardar, buscar, actualizar y borrar objetos — y hacerlo con consultas que se adapten a lo que pida cada usuario, no solo con listados fijos.

---

## 🗃️ El patrón repositorio

Un **repositorio** es una interfaz que representa "el almacén de objetos de un tipo": guardar, buscar por id, listar, borrar — sin que quien lo usa necesite saber cómo está implementado por debajo. Spring Data JPA lleva esta idea al extremo: tú declaras la interfaz (vacía, o casi), y Spring genera la implementación entera en tiempo de ejecución.

```java
public interface AlumnoRepository extends JpaRepository<Alumno, Long> {
    // vacía — y ya tiene save(), findById(), findAll(), delete()...
}
```

---

## 🔧 Las operaciones básicas

| Operación | Qué hace |
|---|---|
| `save(objeto)` | Inserta si el objeto no tiene id (o su id no existe todavía), actualiza si ya existe. |
| `findById(id)` | Busca por identificador — puede no encontrar nada. |
| `findAll()` | Devuelve todos los registros. |
| `delete(objeto)` | Elimina. |

`findById` introduce un matiz importante: **puede no encontrar nada**, y Java necesita una forma de expresar "esto puede no existir" sin recurrir a `null` (fuente clásica de `NullPointerException`). Para eso existe `Optional<T>`: un contenedor que explícitamente puede estar vacío o tener un valor.

```java
Optional<Alumno> resultado = alumnoRepository.findById(id);

Alumno alumno = resultado.orElseThrow(() ->
        new ResponseStatusException(HttpStatus.NOT_FOUND, "Alumno no encontrado"));
```

`orElseThrow(...)` extrae el valor si existe, o lanza la excepción que le indiques si el `Optional` está vacío — ya usaste este patrón sin nombrarlo en el Tema 1, en `findById` de `VideojuegoService`.

---

## 🔍 Por qué las consultas fijas se quedan cortas

Imagina un buscador con varios filtros opcionales: título, precio mínimo, precio máximo, estudio... Cada usuario rellena unos campos y deja otros vacíos. Si intentas resolver esto con métodos de consulta fijos (uno por cada combinación posible de filtros), el número de métodos que necesitarías crece exponencialmente — con solo 4 filtros opcionales, ya son 16 combinaciones posibles.

Una **Specification** (o consulta dinámica) resuelve esto de otra forma: en vez de escribir una consulta completa por combinación, construyes la consulta como **piezas combinables** en tiempo de ejecución — cada filtro es una pieza independiente, y solo se añaden al conjunto final las piezas que corresponden a los filtros que el usuario ha indicado de verdad.

---

## 📄 Qué es la paginación

Ninguna API seria devuelve "todo" en una sola respuesta — si una tabla tiene un millón de filas, no tiene sentido (ni es viable) mandarlas todas de golpe. La **paginación** trocea el resultado en páginas: pides una página concreta, de un tamaño concreto, con un orden concreto, y el servidor te devuelve solo esa porción.

---

## 🎮 Aterrizaje en GameVault: filtros dinámicos de verdad

### El repositorio, con Specifications habilitadas

```java
public interface VideojuegoRepository extends
        JpaRepository<Videojuego, Long>,
        JpaSpecificationExecutor<Videojuego> {
}
```

Con `JpaRepository` ya tienes `save`/`findById`/`findAll` "gratis", sin una sola línea de SQL ni de JDBC — el contraste directo con todo lo que escribiste a mano en el Tema 1. `JpaSpecificationExecutor` añade la capacidad de ejecutar Specifications sobre este repositorio.

### Las Specifications, pieza a pieza

```java
public static Specification<Videojuego> tituloContiene(String titulo) {
    if (titulo == null || titulo.isBlank()) {
        return Specification.unrestricted();
    }
    return (root, query, criteriaBuilder) ->
            criteriaBuilder.like(criteriaBuilder.lower(root.get("titulo")), "%" + titulo.toLowerCase() + "%");
}

public static Specification<Videojuego> precioMayorOIgualA(BigDecimal precioMin) {
    if (precioMin == null) {
        return Specification.unrestricted();
    }
    return (root, query, criteriaBuilder) ->
            criteriaBuilder.greaterThanOrEqualTo(root.get("precio"), precioMin);
}
```

El patrón se repite en cada método: si el filtro es `null` (el usuario no lo ha indicado), devuelve `Specification.unrestricted()` — una Specification "vacía" que no añade ninguna condición al `WHERE`; si el filtro tiene valor, devuelve la condición real (`LIKE` para texto, `greaterThanOrEqualTo` para comparaciones numéricas).

!!! tip "`disponibleEnPlataforma` — de pasada, sin profundizar todavía"
    Verás también una Specification llamada `disponibleEnPlataforma`, que usa una función `jsonb_exists` sobre una columna especial. Ignórala por ahora — trabaja con JSONB, y eso es contenido completo del Tema 3. Aquí solo interesa el patrón general de las demás.

### Combinarlas en el service

```java
@Transactional(readOnly = true)
public Page<VideojuegoResponseDTO> findAllPaginated(VideojuegoFiltroDTO filtro, Pageable pageable) {
    Specification<Videojuego> spec = Specification
            .where(VideojuegoSpecifications.tituloContiene(filtro.titulo()))
            .and(VideojuegoSpecifications.precioMayorOIgualA(filtro.precioMin()))
            .and(VideojuegoSpecifications.precioMenorOIgualA(filtro.precioMax()))
            .and(VideojuegoSpecifications.perteneceAlEstudio(filtro.estudioId()));

    return videojuegoRepository.findAll(spec, pageable)
            .map(this::mapToDTO);
}
```

`Specification.where(...).and(...)` encadena todas las piezas en una sola consulta final — las que son `unrestricted()` no añaden nada, así que si `filtro` llega con todos los campos a `null`, el resultado es "sin condiciones", es decir, todo el catálogo (paginado). `videojuegoRepository.findAll(spec, pageable)` ejecuta esa consulta combinada y, de paso, aplica la paginación: el `Pageable` que recibe el método lleva página, tamaño y orden — la clase concreta de Spring que materializa el concepto de paginación que has visto arriba.

---

## ✏️ Modificar y recuperar objetos persistentes

```java
@Transactional
public VideojuegoResponseDTO update(Long id, VideojuegoCreateDTO dto) {
    Videojuego v = videojuegoRepository.findById(id)
            .orElseThrow(() -> new ResponseStatusException(HttpStatus.NOT_FOUND, "Videojuego no encontrado"));

    v.setTitulo(dto.titulo());
    v.setPrecio(dto.precio());

    return mapToDTO(videojuegoRepository.save(v));
}
```

Este `update` (que ya conoces del Tema 1) es el ejemplo perfecto de "modificar y recuperar objetos persistentes": localizas el objeto con `findById` (pasa a estado *managed*, como viste en el apartado anterior), modificas sus campos directamente sobre el objeto Java, y vuelves a llamar a `save()`. Compáralo con lo que habrías escrito a mano en el Tema 1 con JDBC puro: un `UPDATE ... SET ... WHERE id = ?` completo, con cada columna nombrada explícitamente. Aquí, Hibernate genera ese `UPDATE` por ti a partir de qué campos han cambiado realmente en el objeto.

---

## ✅ Ideas clave

??? tip "Abrir resumen"

    - Un **repositorio** representa el almacén de objetos de un tipo; Spring Data JPA genera su implementación a partir de una interfaz.
    - `Optional<T>` expresa explícitamente "esto puede no existir"; `orElseThrow(...)` extrae el valor o lanza la excepción indicada.
    - Las consultas fijas no escalan con múltiples filtros opcionales — una **Specification** construye la consulta como piezas combinables en tiempo de ejecución.
    - `Specification.unrestricted()` es la pieza "vacía" que se usa cuando un filtro es `null`, para no añadir condición sin necesitar un `if` fuera del método.
    - La **paginación** trocea el resultado en páginas (tamaño, número, orden) — nunca se devuelve "todo" de golpe.
    - `JpaSpecificationExecutor` habilita Specifications sobre un repositorio; `Specification.where(...).and(...)` las combina; `findAll(spec, pageable)` ejecuta la consulta combinada y paginada.
