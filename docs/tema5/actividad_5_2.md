# 🧪 Actividad 5.2: Repaso integrador — un componente sobre el módulo documental

!!! warning "🚧 Contenido pendiente de desarrollo"
    Esta actividad todavía no está redactada. Usa el prompt de más abajo con
    `/improve-notes`, apoyándote en el proyecto **GameVault** adjunto, para generar el
    enunciado definitivo.

---

## Prompt para `/improve-notes`

```text
Redacta la Actividad 5.2 del Tema 5 (RA6 - Componentes de acceso a datos) del módulo
Acceso a Datos (0486), semana real 18 del calendario. Es una actividad de REPASO
INTEGRADOR: aplica el patrón de componente ya trabajado (Actividad 5.1) sobre las otras
tecnologías del módulo. Sigue el patrón de estructura de docs/tema0/actividad_0_6.md y
usa la skill /actividad-plantilla-acceso-a-datos si necesita plantilla/solución en .docx.

IMPORTANTE — enfoque: es una PRÁCTICA GUIADA, no un reto. El alumnado trabaja sobre su
propia copia de GameVault (el mismo proyecto adjunto, construido individualmente durante
el curso). El enunciado debe guiar paso a paso, mostrando el código y explicando cada
decisión; solo se deja sin guiar, como mini-reto, lo que repita un patrón idéntico ya
mostrado — y en esta actividad casi todo repite el patrón de la 5.1, así que aquí el
peso del mini-reto puede ser mayor de lo habitual, siempre con la estructura de
referencia delante.

Objetivo (RA6, criterios f, g): que el alumnado cree en su GameVault un componente
`ReviewsConsultaService` sobre el módulo documental (MongoDB), espejo del
CatalogoConsultaService que ya tiene sobre el módulo relacional — demostrando que el
mismo patrón de componente (interfaz en paquete `api` + implementación oculta) funciona
igual sea cual sea el motor de persistencia por debajo. Este componente NO existe en la
referencia adjunta: es una MEJORA, mencionada como extensión natural en
docs/arquitectura/modulos-y-desacoplamiento.md.

Estructura sugerida de pasos guiados:
1. Planteamiento guiado: ¿qué necesita saber el módulo `catalogo` (u otro futuro) sobre
   las reseñas sin conocer MongoDB? Definir juntos el contrato mínimo: por ejemplo
   `long totalReviewsDe(Long videojuegoId)` y `double puntuacionMediaDe(Long
   videojuegoId)` (la lógica ya existe dentro de ReviewService.getResumenByVideojuegoId
   — se trata de exponerla como componente, no de reescribirla).
2. Mini-reto guiado por la estructura de la 5.1: DEJA EXPLÍCITO en el enunciado, no lo
   des por sobreentendido, que el paquete `reviews.api` NO existe todavía en su
   GameVault (a diferencia de `catalogo.api`, que sí existe en la referencia) — el
   primer paso del mini-reto es crear esa carpeta nueva, por analogía exacta con
   `catalogo/api/`. Después, crear la interfaz en `reviews.api`, la implementación
   package-private anotada `@Service` que reutilice ReviewRepository, y el test aislado
   — el enunciado da la estructura de ficheros esperada (incluida la carpeta nueva) y
   recuerda el patrón, pero el código lo escriben ellos porque es idéntico al de
   CatalogoConsultaService/CatalogoConsultaServiceImpl que ya tienen delante.
3. Uso guiado del componente nuevo desde el módulo `catalogo`: por ejemplo, enriquecer
   VideojuegoResponseDTO (o un endpoint de detalle) con la puntuación media, mostrando
   el código del cambio — y remarcando que `catalogo` no importa nada de
   `reviews` salvo la interfaz del paquete `api`.
4. Verificación guiada: comprobar con una petición real que el dato viaja de MongoDB al
   endpoint del catálogo a través del componente, y que los tests existentes siguen
   pasando.
5. Reflexión de cierre guiada por preguntas concretas: rellenar una tabla comparando sus
   dos componentes (CatalogoConsultaService sobre JPA/PostgreSQL vs.
   ReviewsConsultaService sobre MongoDB): ¿en qué se diferencian sus implementaciones?,
   ¿en qué son idénticas sus interfaces? — la conclusión esperada es que el patrón de
   componente es independiente del motor (criterios f y g cubiertos con el mismo molde).

Esta actividad da paso a la Actividad 5.3, que cierra el módulo completo con la
integración final y las pruebas de todos los componentes juntos.
```
