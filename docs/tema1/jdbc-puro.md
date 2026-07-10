<a id="jdbc-puro"></a>

# 🧩 4. JDBC puro: conexión manual sin ORM

!!! warning "🚧 Contenido pendiente de desarrollo"
    Esta página todavía no tiene la teoría redactada. Usa el prompt de más abajo con
    `/improve-notes`, apoyándote en el proyecto **GameVault** adjunto, para generar el
    contenido definitivo.

---

## Prompt para `/improve-notes`

```text
Redacta el apartado de teoría "JDBC puro: conexión manual sin ORM" del Tema 1 (RA2 -
Manejo de conectores) del módulo Acceso a Datos (0486), semana real 5 del calendario.
Sigue las convenciones de estilo del README.md del repo.

Criterios de evaluación de RA2 que cubre este apartado (curriculum.md):
- a) Ventajas e inconvenientes de utilizar conectores (retómalo aquí desde el ángulo
  contrario al del apartado 1: ahora se ve el coste de NO usar un ORM).
- c) Uso del conector idóneo en la aplicación.
- i) Eliminación de los objetos una vez finalizada su función (cierre manual de
  recursos).

ESTRUCTURA — teoría primero: antes de tocar código, presenta JDBC desde cero como lo
que es: la API estándar de Java para bases de datos (java.sql), la capa sobre la que se
apoya TODO lo demás (incluido Hibernate, que se verá en el Tema 2); sus cuatro piezas
con la función de cada una en una frase (DriverManager/DataSource abre conexiones,
Connection representa la sesión, Statement/PreparedStatement transporta el SQL,
ResultSet es el cursor sobre los resultados); y qué es un "recurso" y por qué los
recursos se cierran (mantienen ocupados una conexión de red y memoria del gestor
mientras viven).

Contenido central: `java.sql.Connection`, `Statement`/`PreparedStatement` y `ResultSet`
manejados a mano, sin Spring Data JPA ni Hibernate de por medio. Es importante que quede
claro que esto es una EXCEPCIÓN deliberada dentro del proyecto: todo GameVault usa
Spring Data JPA (ver com/aleroig/gamevault/catalogo/VideojuegoRepository.java,
EstudioRepository.java, que son simples interfaces JpaRepository sin una sola línea de
JDBC), así que aquí no hay un fichero real de GameVault que mostrar como JDBC puro — el
alumnado va a escribir, en una clase aparte de su GameVault (por ejemplo un
"modo diagnóstico" o un pequeño demo de consola), una consulta manual usando
`DriverManager.getConnection(...)` con la misma URL/usuario/contraseña que ya tiene
configurados en application-dev.yaml para su PostgreSQL de la Actividad 1.1.

Explica el ciclo completo y por qué importa cerrarlo bien:
1. Abrir la conexión (`DriverManager.getConnection` o, mejor, un `DataSource` con pool).
2. Preparar la sentencia con `PreparedStatement` (y por qué NUNCA se debe concatenar el
   SQL con `+` — inyección SQL — usa esto como gancho para explicar los parámetros `?`).
3. Ejecutar y recorrer el `ResultSet`.
4. Cerrar `ResultSet`, `Statement` y `Connection` en orden inverso — o, mejor, con
   try-with-resources — y explica qué pasa si no se cierran (fugas de conexiones,
   agotamiento del pool).

Contrasta explícitamente con lo visto en operaciones-crud-transacciones.md: ahí Spring
gestiona conexión, transacción y cierre por ti (`@Transactional`, `JpaRepository`); aquí
el alumnado ve "la fontanería" que hay debajo, para entender por qué existen los ORM.
No mezcles esto con Hibernate/Spring Data JPA: esos son el Tema 2 (RA3) completo.
```
