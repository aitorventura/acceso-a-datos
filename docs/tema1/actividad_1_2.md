# 🧪 Actividad 1.2: CRUD completo y DTOs sobre el catálogo

!!! warning "🚧 Contenido pendiente de desarrollo"
    Esta actividad todavía no está redactada. Usa el prompt de más abajo con
    `/improve-notes`, apoyándote en el proyecto **GameVault** adjunto, para generar el
    enunciado definitivo.

---

## Prompt para `/improve-notes`

```text
Redacta la Actividad 1.2 del Tema 1 (RA2 - Manejo de conectores) del módulo Acceso a
Datos (0486), semana real 4 del calendario. Sigue el patrón de estructura de
docs/tema0/actividad_0_6.md y usa la skill /actividad-plantilla-acceso-a-datos si la
actividad necesita plantilla/solución en .docx.

IMPORTANTE — enfoque: es una PRÁCTICA GUIADA, no un reto. El alumnado desarrolla su
propia copia de GameVault (el mismo proyecto adjunto, construido individualmente durante
el curso). El enunciado debe guiar paso a paso, mostrando el código y explicando cada
decisión; solo se deja sin guiar, como mini-reto, lo que repita un patrón idéntico ya
mostrado en la misma actividad.

Objetivo (RA2, criterios f, g, h, j): que el alumnado construya, guiado, el CRUD completo
de Videojuego en su GameVault, replicando lo que existe en la referencia:

- com/aleroig/gamevault/catalogo/VideojuegoController.java: GET (lista y por id), POST,
  PUT y DELETE sobre /api/v1/videojuegos.
- com/aleroig/gamevault/catalogo/VideojuegoService.java: la lógica de cada operación,
  con `@Transactional` en las de escritura y `@Transactional(readOnly = true)` en las de
  lectura, y la gestión de "no encontrado" con ResponseStatusException +
  HttpStatus.NOT_FOUND.
- com/aleroig/gamevault/catalogo/dto/: VideojuegoCreateDTO, VideojuegoResponseDTO y el
  mapeo manual mapToDTO() del service (por qué no se devuelve la entidad JPA directa).
  (Nota: en este punto del curso NO incluyas todavía la paginación con Pageable, el
  filtro VideojuegoFiltroDTO ni las Specifications — eso llega en el Tema 2. Tampoco el
  VideojuegoEventPublisher que aparece en el service de referencia: la mensajería es de
  PSP, más adelante; indica en el enunciado que esas líneas se añadirán en su momento.)

Estructura sugerida de pasos guiados:
1. Los DTOs y el mapeo manual, explicados y con código completo.
2. El GET (lista y por id) y el POST, guiados al completo: controller + service, con
   cada anotación explicada.
3. El PUT guiado al completo (es la operación con más matices: cargar, comprobar
   existencia, modificar campos, guardar).
4. El DELETE como mini-reto: repite el patrón ya visto en el PUT (cargar + comprobar +
   actuar), así que basta indicar la firma del endpoint y qué debe devolver (204 No
   Content) y dejar que lo completen solos.
5. Comprobación final guiada con Postman/curl de las cuatro operaciones, con las
   peticiones de ejemplo dadas en el enunciado.
6. Una pregunta de comprensión: ¿qué aporta exactamente @Transactional en create() si
   solo hay una operación de guardado? ¿Y si mañana el método hiciera dos?

Nota de coordinación con PSP (0490): el CRUD de Estudio NO se completa aquí — en el
GameVault de referencia EstudioController solo tiene GET y POST, y el PUT y el DELETE de
Estudio se desarrollan como práctica en PSP (semanas reales 5 y 6, RA4 de PSP). Menciona
esta conexión en una nota del enunciado para que el alumnado entienda que ambos módulos
avanzan sobre el mismo proyecto.
```
