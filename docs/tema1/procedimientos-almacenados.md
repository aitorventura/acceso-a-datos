<a id="procedimientos-almacenados"></a>

# 🧩 5. Procedimientos almacenados

!!! warning "🚧 Contenido pendiente de desarrollo"
    Esta página todavía no tiene la teoría redactada. Usa el prompt de más abajo con
    `/improve-notes`, apoyándote en el proyecto **GameVault** adjunto, para generar el
    contenido definitivo. Este apartado cierra el RA2.

---

## Prompt para `/improve-notes`

```text
Redacta el apartado de teoría "Procedimientos almacenados" del Tema 1 (RA2 - Manejo de
conectores) del módulo Acceso a Datos (0486), semana real 6 del calendario — es el
apartado que CIERRA el RA2. Sigue las convenciones de estilo del README.md del repo.

Criterio de evaluación de RA2 que cubre este apartado (curriculum.md):
- k) Se han ejecutado procedimientos almacenados en la base de datos.

ESTRUCTURA — teoría primero: empieza explicando desde cero qué es un procedimiento
almacenado (código SQL que vive y se ejecuta en el propio motor de base de datos, no en
la aplicación Java), que los gestores tienen un lenguaje procedural propio para
escribirlos (en PostgreSQL, PL/pgSQL: variables, parámetros, control de flujo — muestra
la sintaxis mínima de un CREATE PROCEDURE genérico antes del ejemplo real), la
diferencia entre procedimiento y función, cuándo tiene sentido usarlos (operaciones que
conviene que ocurran atómicamente cerca de los datos, por ejemplo ajustes masivos de
precios) y sus inconvenientes (lógica repartida en dos sitios, más difícil de versionar
y probar). Después, cómo invocarlos desde Java: primero la vía JDBC estándar
(CallableStatement, mencionada), y la que usará el proyecto: `JdbcTemplate`
(`jdbcTemplate.update("CALL ...")` o `SimpleJdbcCall`), explicando qué es JdbcTemplate
(el ayudante de Spring sobre JDBC puro que evita la fontanería de abrir/cerrar vista en
el apartado anterior).

Importante — esto es una MEJORA que NO existe todavía en el GameVault adjunto: revisa
com/aleroig/gamevault/catalogo/EstudioRepository.java y verás que es un JpaRepository
plano, sin ningún procedimiento almacenado ni uso de JdbcTemplate en todo el proyecto.
Escribe el apartado dejando claro que el alumnado va a AÑADIR esta pieza que falta,
siguiendo el espíritu del proyecto (mismo paquete `catalogo`, mismo estilo de código con
Lombok/`@RequiredArgsConstructor` que usa el resto de servicios como
VideojuegoService.java), no a buscarla ya hecha.

Propón como ejemplo guía un procedimiento `ajustar_precio_estudio(estudio_id, porcentaje)`
que suba o baje un porcentaje el precio de todos los Videojuego de un Estudio (aprovecha
que Estudio ya tiene la relación @OneToMany con Videojuego, y que Videojuego.precio es un
BigDecimal con precisión 10,2 — coméntalo para que el alumnado calcule bien el redondeo).
Explica:
1. Cómo se crea el procedimiento almacenado en PostgreSQL (`CREATE OR REPLACE PROCEDURE`
   o función, según el motor — para PostgreSQL 18 usa procedimientos con `CALL`).
2. Cómo se invoca desde Spring con `JdbcTemplate.update("CALL ajustar_precio_estudio(?,
   ?)", estudioId, porcentaje)`, inyectando el `JdbcTemplate` igual que se inyecta un
   `JpaRepository` (con `@RequiredArgsConstructor`, siguiendo el estilo del proyecto).
3. Por qué esta operación tiene sentido como procedimiento almacenado y no como un bucle
   Java que actualice fila a fila (atomicidad, menos idas y vueltas a la BD).

Cierra el apartado recapitulando todo RA2 en 3-4 frases (conectores → CRUD/transacciones
→ JDBC puro → procedimientos almacenados) antes de pasar al Tema 2 (RA3, ORM con
Hibernate), que retoma exactamente donde este tema deja el desfase objeto-relacional.
```
