# 🧪 Actividad 2.3: Ranking con `@Query` JPQL — cierre de RA3

!!! warning "🚧 Contenido pendiente de desarrollo"
    Esta actividad todavía no está redactada. Usa el prompt de más abajo con
    `/improve-notes`, apoyándote en el proyecto **GameVault** adjunto, para generar el
    enunciado definitivo.

---

## Prompt para `/improve-notes`

```text
Redacta la Actividad 2.3 del Tema 2 (RA3 - ORM) del módulo Acceso a Datos (0486), semana
real 9 del calendario — actividad que CIERRA el RA3. Sigue el patrón de estructura de
docs/tema0/actividad_0_6.md y usa la skill /actividad-plantilla-acceso-a-datos si
necesita plantilla/solución en .docx.

IMPORTANTE — enfoque: es una PRÁCTICA GUIADA, no un reto. El alumnado trabaja sobre su
propia copia de GameVault (el mismo proyecto adjunto, construido individualmente durante
el curso). El enunciado debe guiar paso a paso, mostrando el código y explicando cada
decisión; solo se deja sin guiar, como mini-reto, lo que repita un patrón idéntico ya
mostrado en la misma actividad.

Objetivo (RA3, criterios f, g — cierre del RA): que el alumnado añada a su GameVault,
guiado, un ranking de estudios por número de videojuegos con `@Query` JPQL sobre
EstudioRepository (que hoy, en la referencia adjunta, es un JpaRepository plano sin
métodos propios — esto es una MEJORA sobre el proyecto adjunto), expuesto en
`GET /api/v1/estudios/ranking`, siguiendo el estilo de EstudioController.java.

Estructura sugerida de pasos guiados:
1. El método `@Query` en JPQL (no SQL nativo) sobre EstudioRepository, dado y explicado
   parte a parte: `JOIN` sobre la relación, `GROUP BY`, `ORDER BY COUNT(...)` — comparar
   en el propio enunciado la sintaxis JPQL (entidades y propiedades) con el SQL
   equivalente (tablas y columnas).
2. El DTO de respuesta para el ranking (análogo a VideojuegoResponseDTO.java), guiado:
   nombre del estudio y número de videojuegos, explicando por qué NO se expone la lista
   completa de Videojuego de cada Estudio.
3. El método del service con `@Transactional(readOnly = true)` y el endpoint GET en el
   controller, guiados con el código mostrado; pregunta de comprensión: ¿qué ventaja
   concreta aporta readOnly=true frente a no ponerlo?
4. Mini-reto (repite el patrón JPQL recién visto): añadir un criterio de desempate al
   `ORDER BY` (por ejemplo, nombre alfabético) — solo se indica el objetivo — y
   verificarlo con datos de prueba donde el empate ocurra de verdad (el enunciado guía
   cómo crear ese empate).
5. Un cierre que sintetice (2-3 frases del propio alumnado, no copiadas del enunciado)
   cómo esta actividad, junto con las dos anteriores del tema, cubre completo el RA3.

Esta actividad cierra el Tema 2 y da paso al Tema 3 (RA4, JSONB), donde se retomará
VideojuegoSpecifications.disponibleEnPlataforma ya usada de pasada en la Actividad 2.2.
```
