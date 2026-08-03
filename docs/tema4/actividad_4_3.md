# 🧪 Actividad 4.3: Integración final del proyecto

!!! success "Entrega final del módulo"
    Esta es la última actividad evaluable de Acceso a Datos. No es una tarea más — es el cierre de todo lo que has construido desde el Tema 1: catálogo en PostgreSQL, reseñas en MongoDB, componentes desacoplados, y todas las mejoras que has ido añadiendo por el camino.

## Qué vas a entregar

- Un test de integración de extremo a extremo, con Testcontainers, que recorre tu GameVault completo.
- Un documento breve que resuma los componentes que has construido.
- Una autoevaluación personal y razonada.

---

## Requisitos previos

Tu GameVault completo: catálogo (JDBC, JPA, JSONB), reviews (MongoDB), y todos los componentes de este tema.

---

## Paso 1 — El test de integración de flujo completo

No partes de cero: tu `VideojuegoApiIntegrationTest` (Actividad 2.3) ya tiene los tres motores reales conectados —PostgreSQL desde el principio, MongoDB desde la Actividad 4.2, RabbitMQ desde la Actividad 3.1 de PSP— y ya tiene `loginComoAdmin()` y un helper para crear un estudio. No vas a crear ninguna clase nueva ni ningún `@Container` nuevo: añades un `@Test` más a esa misma clase, que encadena todo lo que el resto de tests de ahí ya prueban por separado — crear un videojuego (reutiliza el mismo patrón de `POST /videojuegos` con `detallesPlataforma` que ya ves en los tests existentes) y, a partir de ahí, los tramos que sí son nuevos: reseñas, borrado en cascada, y el registro de actividad que ese mismo borrado dispara en paralelo.

### Tramo 1 — la reseña y el resumen

Sin más código dado, añade un `@Test` nuevo que: haga login como admin y cree un estudio y un videojuego (reutilizando `loginComoAdmin()` y el resto de helpers ya existentes en la clase), cree una reseña para ese videojuego (reutiliza el mismo `adminToken` — crear una reseña solo exige estar autenticado, no rol `ADMIN`, así que te sirve igual), y compruebe con un `GET .../resumen` que el total y la puntuación media son correctos. Repite el patrón que ya usa el resto de tests de la clase (MockMvc completo, `jsonPath` sobre el cuerpo).

### Tramo 2 — el borrado en cascada

Borra el videojuego (de nuevo con `adminToken`, exige rol `ADMIN`), y verifica el borrado en cascada de la Actividad 3.2: la reseña que has creado en el Tramo 1 debe desaparecer de MongoDB. El `RabbitMQContainer` ya entrega el mensaje de verdad, pero sigue siendo asíncrono — necesitas un pequeño margen de espera (un `Thread.sleep` breve, o un mecanismo de espera activa) antes de comprobar que la reseña ya no está.

### Tramo 3 — un evento, dos colas

El borrado que acabas de disparar no tiene un único destino: sobre el mismo exchange (`RabbitMQConfig`, Actividad 3.1 de PSP) hay **dos** colas independientes escuchando `videojuego.eliminado` — la del borrado en cascada de reseñas, que acabas de comprobar, y la del registro de actividad. Amplía el mismo test para comprobar también la segunda: `GET /api/v1/actividad` (con `adminToken`, exige rol `ADMIN`) debe incluir un registro con `tipo: "VIDEOJUEGO_ELIMINADO"` y el `entidadId` del videojuego que has borrado.

**Pregunta**: si solo comprobaras una de las dos colas, ¿qué tipo de fallo de configuración —piensa en el `binding`/`routing key` de `RabbitMQConfig`— podría pasar completamente desapercibido? Relaciónalo con la predicción que has hecho en la teoría de este apartado.

---

## Paso 2 — Documento de cierre

Escribe un documento breve (README de tu propio GameVault, o una sección de memoria si tu centro lo pide así) que liste los componentes que has desarrollado, con 1-2 frases por componente describiendo su responsabilidad. No es un manual exhaustivo, es un mapa rápido de "qué hace cada pieza".

Como mínimo, incluye: `CatalogoConsultaService`, `ReviewsConsultaService`, el flujo de borrado en cascada, el procedimiento almacenado `ajustar_precio_estudio`, el ranking JPQL de estudios.

---

## Paso 3 — Autoevaluación razonada

De todos los temas trabajados en este módulo, ¿con cuál te sientes más flojo, y por qué concretamente? No una respuesta genérica ("necesito practicar más") — señala una pieza técnica exacta con la que todavía no te sientes cómodo, y qué harías para reforzarla.

---

## Paso 4 — Amplía el filtro del CI, y verifica

Antes de comprobar nada, hay algo que arreglar en `.github/workflows/ci.yml`. Ese workflow lo configuraste en la Actividad 1.3 de PSP, y desde entonces filtra los tests con `-Dtest='*ControllerTest'` — una decisión de aquel momento, cuando el único test "pesado" del proyecto era `GamevaultApplicationTests` (el de arranque completo, sin ninguna PostgreSQL real disponible en el runner). Desde entonces has construido muchos más tipos de test que ese filtro nunca ha recogido: los de integración con Testcontainers (Actividad 2.3 de AD, y el que acabas de terminar aquí mismo) y los de componente aislado con Mockito (Actividad 4.1, 4.2) — ninguno de ellos se ha estado ejecutando en tu CI hasta hoy, porque ninguno termina en `ControllerTest`.

Podrías seguir añadiendo sufijos a la lista (`*IntegrationTest`, `*ServiceImplTest`...), pero es una lista que se queda corta cada vez que construyes un tipo de test nuevo — exactamente el problema que acabas de descubrir. Mejor invertir el enfoque: en vez de decir qué **sí** ejecutar, di qué **no** puede ejecutarse en CI (solo `GamevaultApplicationTests`, que no usa Testcontainers y necesita una `dev` que el runner no tiene) y deja que todo lo demás se ejecute por defecto, sea del tipo que sea.

La buena noticia: no hace falta la complejidad que aquella nota de PSP 1.3 anticipaba (levantar servicios manuales en el propio workflow) — los runners de GitHub Actions (`ubuntu-latest`) ya traen Docker instalado, así que Testcontainers puede levantar sus propios contenedores sin tocar el YAML más allá del filtro:

```yaml
      - name: Ejecutar los tests
        run: ./mvnw test -B -Dtest='!GamevaultApplicationTests'
```

El `!` invierte el filtro: en vez de una lista cerrada de qué ejecutar, excluye solo esa clase concreta — todo lo demás (controllers, integración, componentes aislados, y cualquier test nuevo que construyas en el futuro) se ejecuta por defecto, sin que tengas que acordarte de ampliar la lista cada vez.

Haz `push` de tus cambios y comprueba, en la pestaña **Actions** de tu repositorio GitHub, que el pipeline ejecuta correctamente todos tus tests — de controller, de integración y de componente aislado. **Captura**: el resultado en verde del workflow.

---

## ✅ Cierre del módulo

Con esta entrega se cierra Acceso a Datos entero. Tu GameVault ha evolucionado, actividad a actividad, desde un proyecto vacío hasta una aplicación con persistencia relacional, ORM, objeto-relacional, documental y componentes desacoplados — probado de extremo a extremo y verificado automáticamente en cada cambio. Buen trabajo.
