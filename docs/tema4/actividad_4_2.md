# 🧪 Actividad 4.2: Tu propia colección documental

!!! warning "🚧 Contenido pendiente de desarrollo"
    Esta actividad todavía no está redactada. Usa el prompt de más abajo con
    `/improve-notes`, apoyándote en el proyecto **GameVault** adjunto, para generar el
    enunciado definitivo.

---

## Prompt para `/improve-notes`

```text
Redacta la Actividad 4.2 del Tema 4 (RA5 - BD documentales) del módulo Acceso a Datos
(0486), semana real 15 del calendario (11-17 enero). Sigue el patrón de estructura de
docs/tema0/actividad_0_6.md y usa la skill /actividad-plantilla-acceso-a-datos si
necesita plantilla/solución en .docx.

IMPORTANTE — enfoque: es una PRÁCTICA GUIADA, no un reto. El alumnado trabaja sobre su
propia copia de GameVault (el mismo proyecto adjunto, construido individualmente durante
el curso). El enunciado debe guiar paso a paso, mostrando el código y explicando cada
decisión; solo se deja sin guiar, como mini-reto, lo que repita un patrón idéntico ya
mostrado en la misma actividad.

CORRECCIÓN IMPORTANTE sobre el estado real de la referencia: el borrado en cascada de
reseñas al borrar un videojuego YA EXISTE y funciona en el proyecto adjunto, pero de
forma ASÍNCRONA vía RabbitMQ, no como llamada directa. `VideojuegoService.delete()`
publica un evento `VIDEOJUEGO_ELIMINADO` a través de `VideojuegoEventPublisher`, y
`ReviewsVideojuegoEventConsumer` (paquete `reviews.mensajeria`) lo consume y llama a
`reviewService.deleteByVideojuegoId(...)`. `docs/arquitectura/modulos-y-desacoplamiento.md`
documenta explícitamente que la llamada síncrona directa entre `catalogo` y `reviews`
fue el diseño ORIGINAL, descartado a propósito porque acoplaba ambos módulos. NO pidas
al alumnado reintroducir esa llamada síncrona: sería deshacer una decisión de
arquitectura que el propio proyecto documenta como consciente. Antes de construir esta
actividad, confirma que el alumnado ya tiene RabbitMQ explicado como concepto (colas,
exchanges, publicador/consumidor, `@RabbitListener`) — se introduce en PSP,
tema3/hilos-en-gamevault.md (semana real 12); si esta actividad de AD cae antes en el
calendario, remite explícitamente a ese apartado o resume aquí lo mínimo indispensable
para que el alumnado entienda el flujo antes de tocarlo.

Objetivo (RA5, criterios a, d): que el alumnado, sobre su módulo `reviews` (creado en la
Actividad 4.1), entienda y replique en su propio GameVault el flujo real de borrado en
cascada vía eventos, y lo endurezca frente al escenario de fallo a mitad de camino
(idempotencia), que es donde SÍ hay una mejora real que hacer sobre la referencia.

Estructura sugerida de pasos guiados:
1. Lectura guiada del flujo real existente: `VideojuegoService.delete()` →
   `VideojuegoEventPublisher` (publica en RabbitMQ) → `ReviewsVideojuegoEventConsumer`
   (`@RabbitListener`) → `ReviewService.deleteByVideojuegoId(...)`. Diagrama del viaje
   completo del evento, con el énfasis en que el borrado de reseñas ocurre en un hilo
   distinto al de la petición HTTP que originó el borrado (conexión explícita con lo
   visto en PSP sobre hilos de listeners de RabbitMQ).
2. Replicar guiadamente el consumer y el método `deleteByVideojuegoId` en el GameVault
   propio del alumnado (si no los tiene ya de la Actividad 4.1), con el código mostrado
   y explicado.
3. Prueba guiada: crear un videojuego con reseñas, borrarlo desde la API, y comprobar en
   MongoDB (con `mongosh` o Compass, comandos dados) que las reseñas asociadas
   desaparecen tras un pequeño instante (no es instantáneo: es asíncrono) — observar en
   los logs el nombre del hilo del listener frente al hilo de la petición HTTP.
4. LA MEJORA real de esta actividad (a diferencia de la versión anterior de este
   enunciado): hacer `deleteByVideojuegoId` idempotente frente a reintentos o mensajes
   duplicados del broker (por ejemplo, comprobando antes si ya no quedan reseñas, o
   dejando que el propio método no falle si la colección ya está vacía) — código
   mostrado y explicado, con la pregunta de comprensión: ¿qué pasaría si RabbitMQ
   reintenta la entrega del mismo evento dos veces?
5. Reflexión breve de cierre: comparación por escrito entre su experiencia con
   PostgreSQL en los temas anteriores y con MongoDB en este tema, y entre borrar de
   forma síncrona (como haría un JOIN/cascade en un único motor relacional) y de forma
   asíncrona entre dos motores distintos — qué se gana (desacoplamiento) y qué se pierde
   (consistencia inmediata).

Esta actividad da paso a la Actividad 4.3, que cierra RA5 con el PUT de reseñas y control
de autoría.
```
