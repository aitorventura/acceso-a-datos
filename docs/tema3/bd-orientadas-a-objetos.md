<a id="bd-orientadas-a-objetos"></a>

# 🧩 3. Bases de datos orientadas a objetos y cierre de RA4

!!! warning "🚧 Contenido pendiente de desarrollo"
    Esta página todavía no tiene la teoría redactada. Usa el prompt de más abajo con
    `/improve-notes`, apoyándote en el proyecto **GameVault** adjunto, para generar el
    contenido definitivo. Este apartado cierra el RA4.

---

## Prompt para `/improve-notes`

```text
Redacta el apartado de teoría "Bases de datos orientadas a objetos y cierre de RA4" del
Tema 3 (RA4) del módulo Acceso a Datos (0486), semana real 12 del calendario — apartado
que CIERRA el RA4. Sigue las convenciones de estilo del README.md del repo.

Criterios de evaluación de RA4 que cubre este apartado (curriculum.md):
- h) Se han probado y documentado las aplicaciones desarrolladas.
- (contenido de currículo, sin criterio propio de RA4 exactamente pero exigido por el
  temario): gestores de bases de datos orientadas a objetos puras y su interfaz de
  programación de aplicaciones — lenguaje OQL.

Divide el apartado en dos partes claramente diferenciadas:

PARTE 1 — Bloque teórico sobre BD orientadas a objetos puras (sin proyecto asociado,
puesto que GameVault no usa ningún gestor de este tipo — dilo explícitamente para que
quede claro por qué aquí no hay código, a diferencia del resto del tema). Explica el
concepto de BD orientada a objetos pura (los objetos se persisten tal cual, con su
identidad y sus referencias, sin traducirlos a filas/columnas ni siquiera parcialmente
como hace lo objeto-relacional con JSONB) y el lenguaje de consulta OQL (Object Query
Language), comparándolo brevemente con JPQL ya visto en el Tema 2: la diferencia clave es
que en una BD OO pura no hace falta ORM porque el propio motor entiende objetos. Usa
ejemplos históricos/conceptuales conocidos (db4o, ObjectDB, Versant) solo como referencia
de que existen, sin pretender que el alumnado instale ninguno.

PARTE 2 — Pruebas y documentación (criterio h), retomando el proyecto real:
- src/test/java/com/aleroig/gamevault/catalogo/VideojuegoServiceTest.java y
  VideojuegoControllerTest.java: tests unitarios/de capa de GameVault sobre el catálogo
  (con JSONB incluido) — explica la diferencia entre testear el service (lógica, mocks
  de repositorio) y testear el controller (MockMvc, capa HTTP).
- src/test/java/com/aleroig/gamevault/integration/GamevaultApiTest.java: test de
  integración con Testcontainers real (`@Testcontainers`, `PostgreSQLContainer`,
  `@ServiceConnection`, `@AutoConfigureMockMvc`) que levanta un PostgreSQL real en Docker
  solo para la duración del test — explica por qué esto da más confianza que mockear el
  repositorio (se prueba el mapeo JSONB real contra un PostgreSQL real, no una base de
  datos en memoria que podría no soportar `jsonb_exists`).
- Explica "documentar" en el sentido de este RA como comentar en el propio código de test
  qué comportamiento se está verificando y por qué (nombres de test descriptivos), más
  que como documentación externa.

Cierra el apartado recapitulando todo RA4 en 3-4 frases (JSONB como persistencia de
objetos estructurados → consultas jsonb_exists → BD OO puras en teoría → pruebas con
Testcontainers) antes de pasar al Tema 4 (RA5, bases de datos documentales con MongoDB),
que cambia de un modelo objeto-relacional (JSONB dentro de PostgreSQL) a un modelo
documental nativo completo.
```
