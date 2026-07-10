# 🧩 Tema 3: Bases de datos objeto relacionales y orientadas a objetos

> **RA4**: Desarrolla aplicaciones que gestionan la información almacenada en bases de datos objeto relacionales y orientadas a objetos valorando sus características y utilizando los mecanismos de acceso incorporados.

---

## 🎯 Criterios de evaluación

✅ Se han identificado las ventajas e inconvenientes de las bases de datos que almacenan objetos.
✅ Se han establecido y cerrado conexiones.
✅ Se ha gestionado la persistencia de objetos simples.
✅ Se ha gestionado la persistencia de objetos estructurados.
✅ Se han desarrollado aplicaciones que realizan consultas.
✅ Se han modificado los objetos almacenados.
✅ Se han gestionado las transacciones.
✅ Se han probado y documentado las aplicaciones desarrolladas.

---

## 📘 Índice de contenidos

1. [Persistencia de objetos con JSONB](persistencia-objetos-jsonb.md)
2. [Consultas sobre columnas JSONB](consultas-jsonb.md)
3. [Bases de datos orientadas a objetos y cierre de RA4](bd-orientadas-a-objetos.md)

**Actividades:**

- [Actividad 3.1 — La columna `detallesPlataforma`: persistiendo objetos estructurados](actividad_3_1.md)
- [Actividad 3.2 — Filtrar por plataforma con `jsonb_exists`](actividad_3_2.md)
- [Actividad 3.3 — Pruebas de integración sobre JSONB — cierre de RA4](actividad_3_3.md)

---

!!! info "¿Cómo avanzar por el contenido?"
    Utiliza el índice o las flechas de navegación al final de cada página para desplazarte por los distintos apartados de este tema.

!!! note "Nota para el profesorado"
    Este tema corresponde a las semanas reales 10-12 del calendario (RA4, bloque cerrado). Los apartados de teoría y las actividades están pendientes de redactar: cada `.md` de este tema contiene, en su lugar, el prompt que debe usarse con `/improve-notes` para generarlo, apoyándose en el proyecto GameVault adjunto. El criterio b) (establecimiento/cierre de conexiones) se marca como "no aplica" de forma explícita en el calendario porque en un proyecto Spring Data JPA lo gestiona el framework: recuérdalo en la teoría en vez de omitirlo sin más.

    **Evidencia evaluable para el criterio b):** para que no quede sin ninguna evidencia en la entrega final, el criterio se da por cubierto con lo ya demostrado en RA2-d (Tema 1: establecimiento de la conexión vía datasource/HikariCP) más una explicación por escrito del alumnado, en el cierre de RA4 (Actividad 3.3), de por qué JSONB no abre ni cierra una conexión propia — reutiliza la misma conexión JDBC/JPA ya gestionada por el framework. Esa explicación por escrito es la evidencia concreta a pedir en la rúbrica para b), no una implementación de código nueva.
