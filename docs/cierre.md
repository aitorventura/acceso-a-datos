# 🏁 Cierre del módulo

## Lo que has construido

A lo largo de Acceso a Datos, GameVault ha ido ganando una capa de persistencia distinta en cada tema, siempre sobre el mismo proyecto:

| Tema | Qué has añadido |
|---|---|
| **Tema 0** | El entorno de trabajo: Git y GitHub con ramas y Pull Requests, Docker y Dev Container, y tu primer pipeline de CI (sobre un proyecto de práctica, `validador-dni-ci`). |
| **Tema 1** | Spring Boot, JPA/Hibernate para el CRUD de `Videojuego` y `Estudio`, JDBC puro para comparar con el ORM, y un procedimiento almacenado invocado con `JdbcTemplate`. |
| **Tema 2** | Persistencia de objetos con JSONB (`detallesPlataforma`), consultas dinámicas sobre ese campo, y tus primeros tests de integración con Testcontainers. |
| **Tema 3** | MongoDB para las reseñas (`Review`), colecciones documentales, y modificación de documentos con control de autoría. |
| **Tema 4** | Componentes desacoplados (`CatalogoConsultaService`, `ReviewsConsultaService`) sobre los dos motores, y el test de integración final que los prueba juntos. |

Un mismo proyecto, tres motores de persistencia distintos, y un patrón de diseño (interfaz + implementación oculta) que se repite igual sobre cualquiera de los tres.

---

## Huecos reales que quedan (y por qué)

Tu GameVault, tal como queda hoy, no está completo — y no es un descuido. El objetivo del módulo no era que repitieras la misma tarea (un test más, una consulta más, un endpoint más) sobre cada rincón del proyecto hasta agotarlo: era que entendieras cada patrón una vez, lo aplicaras un par de veces con criterio propio, y supieras reconocerlo la próxima vez que haga falta. Eso deja huecos a propósito — estos son reales, verificados sobre tu propio código, no hipotéticos:

- **Tests que no se actualizaron cuando llegó la seguridad**: tu único test con autenticación real, `VideojuegoApiIntegrationTest` (Actividad 2.3), prueba un solo caso `403` (sin rol `ADMIN` en `update` de `Videojuego`) — no cubre `create` ni `delete`, ni ningún endpoint de `Review`. Los tests MockMvc de PSP (`VideojuegoControllerTest`, `EstudioControllerTest`) van en la dirección contraria: usan `@AutoConfigureMockMvc(addFilters = false)`, que desactiva la seguridad por completo, y se escribieron antes de que el proyecto tuviera roles — nunca se han vuelto a tocar desde entonces. `AuthController` no tiene ningún test propio.
- **`documentacion.md`** (la tabla de políticas de seguridad, un fichero compartido con PSP): a día de hoy no incluye las rutas de `Review` que construiste en el Tema 3, ni las de WebSocket de PSP. Un documento de seguridad que no se actualiza al mismo ritmo que el código deja de ser de fiar.
- **El procedimiento almacenado `ajustar_precio_estudio`** (Actividad 1.4): no tiene ninguna documentación fuera del propio código — ni un comentario, ni una entrada en `documentacion.md` — a diferencia de la disciplina de documentar componentes que sí se te pide explícitamente en el Tema 4.

Ninguno de estos huecos es un error que tengas que arreglar para aprobar. Si quieres, son el ejercicio que sigue: coge uno, ciérralo tú mismo aplicando exactamente lo que ya sabes, y comprueba si todavía necesitas que alguien te lo explique paso a paso o si ya te basta con el criterio que has construido durante el curso.

---

## Reflexión final

Has trabajado sobre un proyecto real, no sobre ejercicios sueltos y desconectados entre sí — con las ventajas de eso (el contexto se acumula, las decisiones de un tema condicionan al siguiente) y también con sus inconvenientes (nada queda perfecto ni completo, exactamente como en cualquier proyecto real fuera de clase). Acceso a Datos termina aquí, pero GameVault sigue siendo tuyo: nada te impide, si te interesa, seguir cerrando huecos como los de arriba por tu cuenta.
