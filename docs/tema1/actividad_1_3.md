# 🧪 Actividad 1.3: Acceso a la base de datos con JDBC puro

!!! warning "🚧 Contenido pendiente de desarrollo"
    Esta actividad todavía no está redactada. Usa el prompt de más abajo con
    `/improve-notes`, apoyándote en el proyecto **GameVault** adjunto, para generar el
    enunciado definitivo.

---

## Prompt para `/improve-notes`

```text
Redacta la Actividad 1.3 del Tema 1 (RA2 - Manejo de conectores) del módulo Acceso a
Datos (0486), semana real 5 del calendario. Sigue el patrón de estructura de
docs/tema0/actividad_0_6.md y usa la skill /actividad-plantilla-acceso-a-datos si la
actividad necesita plantilla/solución en .docx.

IMPORTANTE — enfoque: es una PRÁCTICA GUIADA, no un reto. El alumnado trabaja sobre su
propia copia de GameVault (el mismo proyecto adjunto, construido individualmente durante
el curso). El enunciado debe guiar paso a paso, mostrando el código y explicando cada
decisión; solo se deja sin guiar, como mini-reto, lo que repita un patrón idéntico ya
mostrado en la misma actividad.

Objetivo (RA2, criterios a, c, i): que el alumnado escriba, guiado y sin Spring Data JPA
ni Hibernate, una pequeña clase Java que se conecte a su PostgreSQL (el mismo levantado
en la Actividad 1.1) usando `java.sql.Connection`, `PreparedStatement` y `ResultSet` a
mano, y que la compare de forma razonada con el equivalente que ya tiene funcionando vía
Spring Data JPA (VideojuegoRepository.findAll(), que en un `JpaRepository` no requiere ni
una línea de JDBC).

Estructura sugerida de pasos guiados:
1. Una clase de consola (fuera del flujo normal de controladores/servicios, para no
   romper la arquitectura de Spring de su GameVault), guiada al completo con el código
   mostrado y comentado: abrir conexión con DriverManager, ejecutar un PreparedStatement
   parametrizado ("todos los videojuegos de un estudio concreto, pasado su id como
   parámetro") y recorrer el ResultSet mapeando manualmente cada fila a un objeto Java
   (sin usar ninguna anotación JPA).
2. Cierre de recursos con try-with-resources, mostrado en el mismo código y explicado;
   pregunta de comprensión: ¿qué pasaría si no se cerraran? (relacionarlo con el pool de
   conexiones visto en el apartado de teoría de la semana 3).
3. Mini-reto (repite el patrón del paso 1): una segunda consulta parametrizada distinta
   (por ejemplo, "videojuegos con precio inferior a X") reutilizando la misma estructura
   de conexión/statement/resultset ya escrita — solo se da el SQL objetivo, no el código.
4. Una comparación guiada por preguntas concretas entre este código JDBC puro y el
   equivalente con VideojuegoRepository: ¿cuántas líneas hace falta escribir en cada
   caso?, ¿qué se gana y qué se pierde en control fino sobre el SQL exacto ejecutado?
5. Demostración guiada de la inyección SQL: dar el código con `Statement` y concatenación
   y la entrada maliciosa exacta a probar (una comilla simple), pedir que la ejecuten
   contra su base de datos y describan lo observado — el objetivo es verlo pasar de
   verdad, no adivinarlo.

No debe usar Hibernate/JPA en ningún punto: ese enfoque es el Tema 2 (RA3) completo.
```
