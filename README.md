# Acceso a Datos (0486) — Apuntes y actividades

Sitio de apuntes y actividades del módulo **Acceso a Datos** (0486, ciclo DAM), construido con [MkDocs Material](https://squidfunk.github.io/mkdocs-material/). Teoría y actividades giran en torno a un proyecto compartido, **GameVault** (catálogo de videojuegos con Spring Boot), que se va ampliando tema a tema.

## Puesta en marcha

```bash
pip install -r requirements.txt
mkdocs serve
```

Abre la URL que te indique la terminal (por defecto `http://127.0.0.1:8000`).

## Temario

- **Tema 0 — Introducción**: repaso de Git/GitHub y Docker/Dev Containers.
- **Tema 1 — Manejo de conectores y herramientas ORM**: fundamentos de Spring Boot, conectores y protocolos, CRUD y transacciones, JDBC puro, procedimientos almacenados, Hibernate a fondo, consultas dinámicas (Specifications) y consultas JPQL con transacciones.
- **Tema 2 — BD objeto-relacionales y orientadas a objetos**: persistencia y consultas sobre JSONB, bases de datos orientadas a objetos.
- **Tema 3 — BD documentales**: conexión a MongoDB, colecciones documentales, modificación de documentos.
- **Tema 4 — Componentes de acceso a datos**: componentes con conectores/ORM, componentes con BD objeto-relacionales/documentales, integración y pruebas final del proyecto.
