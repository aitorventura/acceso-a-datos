# 🧪 Actividad 5.1: `CatalogoConsultaService` — un componente reutilizable

!!! warning "🚧 Contenido pendiente de desarrollo"
    Esta actividad todavía no está redactada. Usa el prompt de más abajo con
    `/improve-notes`, apoyándote en el proyecto **GameVault** adjunto, para generar el
    enunciado definitivo.

---

## Prompt para `/improve-notes`

```text
Redacta la Actividad 5.1 del Tema 5 (RA6 - Componentes de acceso a datos) del módulo
Acceso a Datos (0486), semana real 17 del calendario. Sigue el patrón de estructura de
docs/tema0/actividad_0_6.md y usa la skill /actividad-plantilla-acceso-a-datos si
necesita plantilla/solución en .docx.

IMPORTANTE — enfoque: es una PRÁCTICA GUIADA, no un reto. El alumnado trabaja sobre su
propia copia de GameVault (el mismo proyecto adjunto, construido individualmente durante
el curso). El enunciado debe guiar paso a paso, mostrando el código y explicando cada
decisión; solo se deja sin guiar, como mini-reto, lo que repita un patrón idéntico ya
mostrado en la misma actividad.

Objetivo (RA6, criterios a, b, d, e): que el alumnado analice a fondo, guiado, el
componente que ya construyó en la Actividad 4.1 —
com/aleroig/gamevault/catalogo/api/CatalogoConsultaService.java (interfaz) +
CatalogoConsultaServiceImpl.java (implementación) — ahora bajo la óptica de
"componente" presentada en la teoría, y lo extienda con un método nuevo.

Estructura sugerida de pasos guiados:
1. Relectura guiada del componente existente en su GameVault, con preguntas dirigidas
   sobre el propio código: ¿por qué la interfaz vive en el paquete `catalogo.api` y la
   implementación no?, ¿por qué CatalogoConsultaServiceImpl no lleva el modificador
   `public`?, ¿qué inyecta ReviewService: la interfaz o la implementación? — apoyándose
   en docs/arquitectura/modulos-y-desacoplamiento.md de GameVault como lectura guiada.
2. Explicación guiada del desacoplamiento con un caso concreto: si mañana cambiara la
   forma de almacenar el catálogo (se sustituyera JPA por otra tecnología), ¿qué ficheros
   habría que tocar y cuáles no? — el enunciado recorre la respuesta con el alumnado.
3. Mini-reto (repite el patrón exacto del componente existente): añadir un segundo
   método a la interfaz, por ejemplo `tituloDe(Long videojuegoId)`, implementarlo en
   CatalogoConsultaServiceImpl y usarlo desde ReviewService (por ejemplo, para
   enriquecer el resumen de reseñas con el título) — solo se indica el objetivo; la
   estructura ya la conocen.
4. Prueba guiada del componente aislado, siguiendo el patrón de
   src/test/java/com/aleroig/gamevault/catalogo/CatalogoConsultaServiceImplTest.java de
   la referencia (código mostrado y explicado).
5. Pregunta de comprensión: ¿qué información NO deberías exponer nunca a través de este
   componente (por ejemplo, la entidad Videojuego completa)? ¿Dónde está la línea entre
   "lo mínimo necesario" y "exponer todo por comodidad"?

Esta actividad da paso a la Actividad 5.2, donde se aplicará el mismo patrón de
componente sobre el módulo documental (MongoDB).
```
