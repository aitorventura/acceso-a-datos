# 🧪 Actividad 2.1: Mapeo con anotaciones — qué SQL genera Hibernate por debajo

!!! info "Práctica guiada — experimentos sobre tu propio mapeo"
    No vas a escribir entidades nuevas hoy — vas a experimentar sobre las que ya tienes (`Videojuego`, `Estudio`), cambiando anotaciones a propósito y observando qué SQL genera Hibernate en cada caso.

## Qué vas a practicar

- Entender por qué `GenerationType.IDENTITY` encaja con PostgreSQL.
- Comprobar experimentalmente el efecto de `@Column(precision, scale)`.
- Comparar `FetchType.LAZY` frente a `EAGER` mirando el SQL real generado.
- Comprobar el efecto de `cascade`/`orphanRemoval` al borrar.

---

## Requisitos previos

Tus entidades `Videojuego` y `Estudio` del Tema 1, con algunos datos de prueba ya creados.

---

## Paso 1 — `GenerationType.IDENTITY`, ¿por qué esa estrategia?

Mira tu anotación actual:

```java
@Id
@GeneratedValue(strategy = GenerationType.IDENTITY)
private Long id;
```

Existen otras estrategias (`SEQUENCE`, `AUTO`, `TABLE`) — `IDENTITY` delega la generación del id directamente en una columna autoincremental de la base de datos (en PostgreSQL, por debajo usa un tipo serial/identity nativo), mientras que `SEQUENCE` usa un objeto secuencia independiente que Hibernate puede consultar por adelantado (útil para reservar varios ids antes de insertar, algo que `IDENTITY` no permite).

**Pregunta**: dado que PostgreSQL soporta columnas identity de forma nativa y eficiente, ¿por qué crees que el proyecto ha elegido `IDENTITY` en vez de `SEQUENCE`, si ambas son válidas?

---

## Paso 2 — Experimento con `@Column(precision, scale)`

Con tu aplicación **parada**, quita temporalmente la anotación de `precio`:

```java
// Antes
@Column(precision = 10, scale = 2)
private BigDecimal precio;

// Ahora, para el experimento
private BigDecimal precio;
```

Borra la tabla `videojuego` de tu base de datos (o usa una base de datos de prueba nueva) y arranca la aplicación con `show-sql: true` activo. Mira en la consola el `create table` generado, y comprueba el tipo de columna con:

```bash
docker exec -it <tu-contenedor-postgres> psql -U gamevault_user -d gamevault_db -c "\d videojuego"
```

**Anota** el tipo exacto de la columna `precio` sin la anotación.

Ahora vuelve a poner `@Column(precision = 10, scale = 2)`, borra la tabla de nuevo, reinicia, y repite la comprobación.

**Responde**: ¿qué diferencia exacta hay entre los dos tipos de columna generados? ¿Qué problema práctico podría causar la versión sin precisión definida, en una columna que guarda dinero?

---

## Paso 3 — `LAZY` vs. `EAGER`, mirando el SQL real

Activa el log detallado de Hibernate en `application-dev.yaml` si no lo tienes ya:

```yaml
logging:
  level:
    org.hibernate.SQL: DEBUG
```

Con tu relación actual (`fetch = FetchType.LAZY` en `Videojuego.estudio`), llama a tu endpoint `GET /api/v1/videojuegos` y mira los logs. **Anota** cuántas consultas SQL distintas ves.

Cambia temporalmente a `fetch = FetchType.EAGER`, reinicia, repite la misma petición, y vuelve a mirar los logs.

**Compara** ambos casos: ¿el número de consultas SQL es el mismo, o cambia? ¿En qué momento (antes o después de necesitar realmente el dato de `Estudio`) se ejecuta la consulta relacionada en cada caso? Deja tu código con `LAZY` al terminar — es la configuración correcta del proyecto.

---

## Paso 4 — El experimento de cierre: cascade y orphanRemoval

Con tu configuración actual (`cascade = CascadeType.ALL, orphanRemoval = true` en `Estudio.videojuegos`), crea un `Estudio` con dos o tres `Videojuego` asociados. Bórralo:

```bash
curl -X DELETE http://localhost:8080/api/v1/estudios/{id}
```

**Comprueba** en la base de datos que los videojuegos asociados también han desaparecido.

Ahora, quita temporalmente `cascade = CascadeType.ALL, orphanRemoval = true` de `Estudio.videojuegos` (déjalo solo como `@OneToMany(mappedBy = "estudio")`), reinicia, crea otro estudio con videojuegos, e intenta borrarlo de la misma forma.

**Observa** qué pasa esta vez (probablemente un error, por la restricción de clave foránea que sigue apuntando a un estudio que ya no existe).

**Pregunta de comprensión**: ¿por qué la cascada se declara en el lado `Estudio → Videojuego` de la relación, y no tendría sentido ponerla al revés (`Videojuego → Estudio`)? Piensa en qué significaría, en ese caso, borrar un solo videojuego. Cuando termines, restaura `cascade = CascadeType.ALL, orphanRemoval = true` — es la configuración correcta del proyecto.

---

## ✅ Cierre

Has comprobado con SQL real, no solo de memoria, qué hace cada decisión de mapeo que ya tenías en tu código desde el Tema 1. En el siguiente apartado dejas de mirar el mapeo y empiezas a trabajar con consultas dinámicas: Specifications.
