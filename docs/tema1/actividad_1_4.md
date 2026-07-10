# 🧪 Actividad 1.4: Procedimiento almacenado con `JdbcTemplate` — cierre de RA2

!!! warning "🚧 Contenido pendiente de desarrollo"
    Esta actividad todavía no está redactada. Usa el prompt de más abajo con
    `/improve-notes`, apoyándote en el proyecto **GameVault** adjunto, para generar el
    enunciado definitivo.

---

## Prompt para `/improve-notes`

```text
Redacta la Actividad 1.4 del Tema 1 (RA2 - Manejo de conectores) del módulo Acceso a
Datos (0486), semana real 6 del calendario — actividad que CIERRA el RA2. Sigue el patrón
de estructura de docs/tema0/actividad_0_6.md y usa la skill
/actividad-plantilla-acceso-a-datos si necesita plantilla/solución en .docx.

IMPORTANTE — enfoque: es una PRÁCTICA GUIADA, no un reto. El alumnado trabaja sobre su
propia copia de GameVault (el mismo proyecto adjunto, construido individualmente durante
el curso). El enunciado debe guiar paso a paso, mostrando el código y explicando cada
decisión; solo se deja sin guiar, como mini-reto, lo que repita un patrón idéntico ya
mostrado en la misma actividad.

Objetivo (RA2, criterio k — cierre del RA): que el alumnado implemente en su GameVault
el procedimiento almacenado `ajustar_precio_estudio(estudio_id, porcentaje)` descrito en
el apartado de teoría procedimientos-almacenados.md (una MEJORA que no existe en la
referencia adjunta), y lo exponga a través de un endpoint REST nuevo, por ejemplo
`POST /api/v1/estudios/{id}/ajustar-precio` (usa solo verbos ya presentes en el
proyecto: GET/POST/PUT/DELETE — NO uses PATCH), siguiendo el estilo de
EstudioController.java/EstudioService.java ya existentes en GameVault (mismo paquete
`catalogo`, mismas convenciones: DTO de entrada con el porcentaje, `ResponseStatusException`
para "estudio no encontrado", inyección con `@RequiredArgsConstructor`).

Estructura sugerida de pasos guiados:
1. El SQL del procedimiento almacenado en PostgreSQL, dado y explicado línea a línea:
   actualiza el precio de todos los Videojuego de un Estudio en una sola llamada (no un
   bucle de updates individuales desde Java). Indicar también dónde/cómo crearlo (psql,
   DBeaver o script de inicialización).
2. La invocación desde el service con `JdbcTemplate`, guiada al completo: inyección
   junto al resto de repositorios, `jdbcTemplate.update("CALL ...", ...)`, y el endpoint
   POST en el controller con su DTO de entrada.
3. Comprobación guiada del efecto antes/después (con psql o DBeaver) sobre los datos de
   prueba que ya carga `DataInitializer.java`/`datos_iniciales.json`, con las consultas
   de verificación dadas en el enunciado.
4. Mini-reto (repite el patrón ya visto): añadir al procedimiento una salvaguarda para
   que ningún precio quede por debajo de 0 (por ejemplo con `GREATEST`), dando solo la
   pista de la función SQL a usar.
5. Una pregunta de comprensión de cierre: repasar, en 3-4 frases propias, cómo esta
   actividad conecta con las tres anteriores del tema (conector → CRUD/transacciones →
   JDBC puro → procedimiento almacenado), como resumen de cierre de todo RA2.

No introduzcas ORM/Hibernate en detalle en esta actividad (eso es el Tema 2 completo);
el procedimiento se invoca con JdbcTemplate, no con una entidad JPA.
```
