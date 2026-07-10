<a id="modificacion-documentos"></a>

# 🧩 3. Modificación de documentos y cierre de RA5

!!! warning "🚧 Contenido pendiente de desarrollo"
    Esta página todavía no tiene la teoría redactada. Usa el prompt de más abajo con
    `/improve-notes`, apoyándote en el proyecto **GameVault** adjunto, para generar el
    contenido definitivo. Este apartado cierra el RA5.

---

## Prompt para `/improve-notes`

```text
Redacta el apartado de teoría "Modificación de documentos y cierre de RA5" del Tema 4
(RA5 - BD documentales) del módulo Acceso a Datos (0486), semana real 16 del calendario
(18-24 enero) — apartado que CIERRA el RA5. Sigue las convenciones de estilo del
README.md del repo.

Criterio de evaluación de RA5 que cubre este apartado (curriculum.md):
- e) Se han desarrollado aplicaciones para añadir, modificar y eliminar documentos de la
  base de datos.

ESTRUCTURA — teoría primero: antes del caso del proyecto, explica desde cero cómo se
añaden, modifican y eliminan documentos en una base de datos documental, con los
comandos genéricos de mongosh mostrados (insertOne, updateOne con $set, replaceOne,
deleteOne) y la diferencia conceptual entre actualizar campos concretos ($set) y
reemplazar el documento entero — porque Spring Data (save()) hace lo segundo, y conviene
que el alumnado sepa que lo primero existe aunque el proyecto no lo use.

Contenido central: en el estado actual de GameVault,
com/aleroig/gamevault/reviews/VideojuegoReviewController.java solo expone `GET` (listar
reseñas), `POST` (crear reseña) y `GET .../resumen` — NO existe todavía un `PUT` para
modificar una reseña ya publicada. Explica esto como una pieza que falta y que el
alumnado va a añadir (MEJORA sobre el proyecto adjunto), con una particularidad
importante: **control de autoría** — solo el autor de una reseña debería poder
modificarla, no cualquier usuario autenticado.

Apóyate en el proyecto GameVault (com.aleroig.gamevault) como ejemplo real:
- com/aleroig/gamevault/reviews/VideojuegoReviewController.java (método create): ya usa
  `Principal principal` para tomar `principal.getName()` como autor de la reseña al
  crearla vía POST — explica que `Principal` es la identidad del usuario autenticado que
  inyecta Spring Security (trabajado en detalle en el módulo de Programación de Servicios
  y Procesos, RA5, semanas 7-11: JWT, roles, rutas protegidas — remite a esa referencia
  cruzada sin explicar JWT en profundidad aquí, ya que no es contenido de Acceso a Datos).
- com/aleroig/gamevault/reviews/Review.java: el campo `autor` (String) es lo que hay que
  comparar contra `principal.getName()` antes de permitir la modificación — explica el
  patrón: "cargar el documento por id, comprobar que el autor coincide con el usuario
  autenticado, y solo entonces aplicar los cambios y guardar" (si no coincide, devolver
  403 Forbidden, no 404, para no dar pistas de que el recurso existe... aunque puedes
  simplificarlo a 403 sin entrar en ese matiz de seguridad si complica el nivel).
- ReviewRepository (MongoRepository<Review, String>) ya tiene `save()` heredado, que
  sirve tanto para crear como para actualizar un documento existente (basta con que el
  objeto tenga el mismo `id`) — remarca la diferencia con JPA/Hibernate donde
  `save()` también sirve para ambos casos, así que el concepto no es nuevo, solo el
  motor de base de datos por debajo.

Cierra el apartado recapitulando todo RA5 en 3-4 frases (conexión y consultas MongoDB →
comparación relacional/documental y colecciones → modificación de documentos con control
de autoría) antes de pasar al Tema 5 (RA6, componentes de acceso a datos), que retomará
`CatalogoConsultaService`/`CatalogoConsultaServiceImpl` (ya usados en este mismo tema para
la integridad referencial manual) como el ejemplo central de "componente".
```
