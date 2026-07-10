# 🧪 Actividad 3.1: La columna `detallesPlataforma` — persistiendo objetos estructurados

!!! warning "🚧 Contenido pendiente de desarrollo"
    Esta actividad todavía no está redactada. Usa el prompt de más abajo con
    `/improve-notes`, apoyándote en el proyecto **GameVault** adjunto, para generar el
    enunciado definitivo.

---

## Prompt para `/improve-notes`

```text
Redacta la Actividad 3.1 del Tema 3 (RA4 - BD objeto relacionales y orientadas a objetos)
del módulo Acceso a Datos (0486), semana real 10 del calendario. Sigue el patrón de
estructura de docs/tema0/actividad_0_6.md y usa la skill
/actividad-plantilla-acceso-a-datos si necesita plantilla/solución en .docx.

IMPORTANTE — enfoque: es una PRÁCTICA GUIADA, no un reto. El alumnado trabaja sobre su
propia copia de GameVault (el mismo proyecto adjunto, construido individualmente durante
el curso). El enunciado debe guiar paso a paso, mostrando el código y explicando cada
decisión; solo se deja sin guiar, como mini-reto, lo que repita un patrón idéntico ya
mostrado en la misma actividad.

Objetivo (RA4, criterios a, c, d): que el alumnado añada a la entidad Videojuego de su
GameVault la columna JSONB `detallesPlataforma`, tal y como existe en
com/aleroig/gamevault/catalogo/Videojuego.java, usando exactamente las mismas
anotaciones (`@JdbcTypeCode(SqlTypes.JSON)`, `@Column(columnDefinition = "jsonb")`, tipo
`Map<String, Object>`), y la exponga en su API (POST/PUT ya existentes, ampliando
VideojuegoCreateDTO/VideojuegoResponseDTO de forma análoga a la referencia).

Estructura sugerida de pasos guiados:
1. Añadir el campo a la entidad y ampliar los DTOs, guiado con el código mostrado y cada
   anotación explicada (incluida la configuración de mapeo JSON que necesite Hibernate,
   siguiendo config/Jackson3HibernateMapper.java de la referencia si aplica).
2. Crear guiadamente un videojuego con `detallesPlataforma` de una sola plataforma
   (objeto simple dentro del JSON), con la petición de ejemplo dada; después, como
   mini-reto que repite el mismo patrón, crear otro con varias plataformas anidadas
   (objeto estructurado) eligiendo ellos las claves.
3. Verificación guiada con una consulta SQL directa (psql/DBeaver): comprobar que
   PostgreSQL ha guardado realmente el tipo `jsonb` y no un simple `text`, con el
   comando exacto indicado (`\d videojuego` o equivalente) y pegando el resultado.
4. Experimento guiado: actualizar `detallesPlataforma` con un PUT (petición de ejemplo
   dada) y observar que sustituye el objeto completo, no lo combina con el anterior;
   pregunta de comprensión sobre por qué ocurre (mapeo directo Map→JSONB, sin merge
   automático).
5. Pregunta de comprensión final: ¿qué ventaja concreta tiene JSONB frente a una tabla
   `plataforma_videojuego` con una fila por plataforma, y en qué caso sería mejor la
   tabla relacional? (para que no quede la idea de que JSONB es "siempre mejor").

Esta actividad no debe usar `jsonb_exists` ni Specifications sobre el JSONB: eso es la
Actividad 3.2, siguiente en este mismo tema.
```
