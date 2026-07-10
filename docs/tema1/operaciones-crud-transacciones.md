<a id="operaciones-crud-transacciones"></a>

# 🧩 3. Operaciones CRUD y gestión de transacciones

!!! warning "🚧 Contenido pendiente de desarrollo"
    Esta página todavía no tiene la teoría redactada. Usa el prompt de más abajo con
    `/improve-notes`, apoyándote en el proyecto **GameVault** adjunto, para generar el
    contenido definitivo.

---

## Prompt para `/improve-notes`

```text
Redacta el apartado de teoría "Operaciones CRUD y gestión de transacciones" del Tema 1
(RA2 - Manejo de conectores) del módulo Acceso a Datos (0486), semana real 4 del
calendario. Sigue las convenciones de estilo del README.md del repo y de
docs/tema0/introduccion-docker.md.

Criterios de evaluación de RA2 que cubre este apartado (curriculum.md):
- f) Aplicaciones que modifican el contenido de la base de datos.
- g) Objetos destinados a almacenar el resultado de las consultas.
- h) Aplicaciones que efectúan consultas.
- j) Gestión de las transacciones.

ESTRUCTURA OBLIGATORIA — teoría primero, proyecto después.

PARTE 1 — Teoría general, desde cero y sin mencionar GameVault todavía:
- Qué significa CRUD (Create, Read, Update, Delete): las cuatro operaciones universales
  sobre datos y su correspondencia con el SQL que ya conocen (INSERT, SELECT, UPDATE,
  DELETE).
- Qué es una transacción, desde cero: unidad de trabajo todo-o-nada, con el ejemplo
  clásico de la transferencia bancaria (restar de una cuenta y sumar en otra: si falla a
  mitad, desastre); commit y rollback; las propiedades ACID explicadas en una frase cada
  una, sin siglas sueltas.
- Qué es un DTO (objeto de transferencia de datos) como concepto general: un objeto
  hecho a medida de lo que entra o sale de la aplicación, distinto del objeto interno —
  y por qué separar ambos (control de qué se expone, formas distintas para crear y para
  leer).
- IMPRESCINDIBLE explicar aquí, antes de mostrar ningún DTO real: qué es un `record` de
  Java (desde Java 16), porque los DTOs de GameVault están implementados como records y
  esta es la PRIMERA vez en todo el curso (AD y PSP) que el alumnado se los encuentra —
  ninguna otra página del curso lo explica, así que esta es la referencia. Un ejemplo
  mínimo genérico primero (`record Punto(int x, int y) {}`) mostrando qué genera el
  compilador solo con esa línea (constructor, getters sin prefijo `get`, `equals`,
  `hashCode`, `toString`) y por qué encaja tan bien con un DTO: es inmutable (todos sus
  campos son `final`, no hay setters) y no necesita una clase tradicional de 20 líneas
  con Lombok para conseguir lo mismo. Aclara que un record NO es lo mismo que una
  `@Entity` (que sí necesita mutabilidad y un id gestionado por Hibernate) — por eso las
  entidades JPA siguen siendo clases normales y los DTOs, records.

PARTE 2 — Aterrizaje en GameVault (com.aleroig.gamevault):
- com/aleroig/gamevault/catalogo/VideojuegoController.java: expone GET (lista paginada
  con filtros y GET por id), POST, PUT y DELETE sobre /api/v1/videojuegos — un CRUD
  completo real que puedes usar tal cual como ejemplo de "aplicaciones que modifican y
  consultan el contenido de la base de datos".
- com/aleroig/gamevault/catalogo/VideojuegoService.java: usa `@Transactional` en los
  métodos de escritura y `@Transactional(readOnly = true)` en los de solo lectura —
  explica aquí qué es una transacción, por qué el commit/rollback automático de Spring
  evita tener que gestionar manualmente `connection.commit()`/`rollback()`, y contrasta
  con lo que se verá "a mano" en el siguiente apartado (jdbc-puro.md).
- com/aleroig/gamevault/catalogo/dto/VideojuegoResponseDTO.java,
  VideojuegoCreateDTO.java y VideojuegoFiltroDTO.java: son el ejemplo de "objetos
  destinados a almacenar el resultado de las consultas" (DTOs) — explica por qué no se
  devuelve la entidad JPA directamente al cliente (acoplamiento, exposición de columnas
  internas, problemas de serialización con relaciones lazy).
- Contrasta con com/aleroig/gamevault/catalogo/EstudioController.java, que en el estado
  actual de GameVault SOLO tiene GET y POST (sin PUT ni DELETE) — coméntalo como ejemplo
  de CRUD incompleto, y aclara que ese PUT y DELETE de Estudio se desarrollarán como
  práctica en el módulo de PSP (0490, semanas reales 5 y 6), no aquí: ambos módulos
  avanzan sobre el mismo GameVault y se reparten el trabajo.

Pide que la explicación conecte con la actividad 1.2, donde el alumnado construirá,
guiado, el CRUD completo de Videojuego en su GameVault aplicando `@Transactional`
correctamente. No entres en JDBC puro ni en procedimientos almacenados: eso va en los
apartados siguientes.
```
