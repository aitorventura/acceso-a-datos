<a id="conexion-mongodb"></a>

# 🧩 1. Conexión a MongoDB y consultas sobre reseñas

Hasta ahora, todo lo que has persistido ha vivido dentro de PostgreSQL — incluso el JSONB del Tema 3 seguía siendo una columna de una tabla relacional. Este tema da un salto real: una base de datos completamente distinta, sin tablas de ningún tipo.

---

## 🗂️ Qué es NoSQL

**NoSQL** es el nombre que se le da a la familia de bases de datos que abandonan el modelo tabla/fila/SQL en favor de otros modelos de datos. Las principales categorías:

- **Documentales**: documentos autocontenidos, tipo JSON (donde se sitúa este tema).
- **Clave-valor**: pares simples clave → valor (como Redis, que ya conoces del Tema 3 de PSP).
- **Columnares**: optimizadas para leer columnas completas de grandes volúmenes de datos.
- **De grafos**: optimizadas para relaciones muy conectadas (nodos y aristas).

---

## 📄 Bases de datos documentales nativas: documento y colección

Dos piezas, y ninguna tabla:

- **Documento**: un objeto completo, tipo JSON, autocontenido — con su propia estructura, que puede variar de un documento a otro dentro de la misma colección.
- **Colección**: un grupo de documentos, sin esquema fijo impuesto por el motor.

Contraste explícito con lo relacional: no hay tablas, ni columnas fijas, ni claves foráneas, ni `JOIN`. Y contraste con el JSONB del Tema 3: allí, el JSON era **una columna** dentro de una tabla relacional normal — aquí, **todo** es documento, no hay ninguna tabla por debajo.

---

## 🍃 Qué es MongoDB

**MongoDB** es el gestor documental más usado. Sus documentos se almacenan en **BSON** (una versión binaria de JSON), y su identificador es un `ObjectId` — alfanumérico, no un número autoincremental como los que has usado hasta ahora en PostgreSQL. Se organiza en niveles: servidor → bases de datos → colecciones → documentos.

Un documento de ejemplo, y su consulta equivalente en `mongosh` (la consola de MongoDB, análoga a `psql`):

```javascript
// Un documento
{ "_id": ObjectId("..."), "titulo": "El nombre del viento", "puntuacion": 9 }

// El equivalente a un SELECT sencillo
db.resenas.find({ "titulo": "El nombre del viento" })
```

Vale la pena verlo en esta forma cruda antes de que Spring lo esconda tras un repositorio.

---

## ⚖️ Cuándo encaja lo documental (intuición)

Datos autocontenidos, de estructura variable de un registro a otro: bien. Relaciones fuertes entre entidades y transacciones complejas que las involucren: mejor relacional. Profundizarás en esta comparación en el siguiente apartado — de momento, quédate con la intuición.

---

## 📚 Un ejemplo completo: las reseñas de la librería

### Por qué dos bases de datos distintas

Siguiendo con la librería: el catálogo (`Libro`/`Editorial`) vive en PostgreSQL — relaciones claras, necesidad de transacciones ACID fuertes. Pero imagina que ahora los lectores pueden dejar **reseñas** de los libros: documentos independientes entre sí, sin relaciones que mantener, con forma que podría evolucionar con el tiempo. Eso encaja mejor en **MongoDB**. Usar dos motores a la vez es una decisión de arquitectura habitual, no una moda — cada uno se usa donde encaja mejor.

### La entidad `Resena`

```java
@Document(collection = "resena")
public class Resena {

    @Id
    private String id; // alfanumérico, tipo ObjectId — no un Long autogenerado

    private Long libroId; // relación "lógica" con el catálogo en PostgreSQL
    private String autor;
    private Integer puntuacion;
    private String comentario;
}
```

`@Document(collection = "resena")` es el equivalente Mongo de `@Entity`/`@Table` — declara en qué colección vive. El `@Id` es `String`, no `Long`: en Mongo los identificadores son alfanuméricos por naturaleza (`ObjectId`), a diferencia del autoincremental que has usado siempre en PostgreSQL.

!!! warning "`libroId` no es una clave foránea real"
    Aunque el campo se llame `libroId` y apunte conceptualmente a un `Libro` de PostgreSQL, **no hay ninguna integridad referencial automática** entre dos motores de base de datos distintos. MongoDB no sabe nada de PostgreSQL, ni al revés — es responsabilidad de tu código mantener esa relación con sentido.

### El repositorio, con naming de método (igual que en JPA)

```java
public interface ResenaRepository extends MongoRepository<Resena, String> {
    List<Resena> findByLibroId(Long libroId);
}
```

Spring Data también genera consultas automáticas por el nombre del método en MongoDB, exactamente igual que hacía `JpaRepository` en el Tema 2 — sin que escribas una sola query explícita.

### El patrón de "integridad referencial manual"

```java
public List<ResenaResponseDTO> findByLibroId(Long libroId) {
    // 1. Verificamos que el libro exista en PostgreSQL antes de buscar sus reseñas
    if (!libroRepository.existsById(libroId)) {
        throw new ResponseStatusException(HttpStatus.NOT_FOUND, "Libro no encontrado en el catálogo");
    }
    // 2. Buscamos en MongoDB y mapeamos
    return resenaRepository.findByLibroId(libroId).stream().map(this::mapToDTO).toList();
}
```

Este es el patrón central de este apartado: como Mongo no puede validar una clave foránea hacia una tabla de PostgreSQL, el propio código de la aplicación tiene que hacerlo — comprobando primero contra PostgreSQL (`existsById`, Spring Data JPA) antes de tocar Mongo.

### Una agregación sencilla, en memoria

```java
public ResenaResumenDTO getResumenByLibroId(Long libroId) {
    List<Resena> resenas = resenaRepository.findByLibroId(libroId);
    long totalResenas = resenas.size();
    double puntuacionMedia = resenas.stream().mapToInt(Resena::getPuntuacion).average().orElse(0.0);
    return new ResenaResumenDTO(libroId, totalResenas, puntuacionMedia);
}
```

`getResumenByLibroId` calcula el total y la media de puntuación **en memoria**, con streams de Java, tras traer todos los documentos de Mongo — no usa el framework de agregación nativo de Mongo (`$group`, `$avg`). Es una simplificación deliberada: funciona bien con pocos documentos por libro, y queda como posible mejora si algún día hubiera muchísimas reseñas por consultar.

### Establecer la conexión

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-mongodb</artifactId>
</dependency>
```

```yaml
spring:
  mongodb:
    uri: mongodb://localhost:27017/libreria_db
```

!!! warning "La propiedad real es `spring.mongodb.uri`, no `spring.data.mongodb.uri`"
    Es fácil confundirse con tutoriales antiguos: en la versión de Spring Boot que usas en este curso, `mongodb` cuelga directamente de `spring`, no de `spring.data`. Comprueba siempre tu `application-dev.yaml` antes de dar por buena una propiedad.

En paralelo al `datasource` de PostgreSQL que ya conoces del Tema 1, esta es toda la configuración necesaria para conectar con MongoDB.

---

## ✅ Ideas clave

??? tip "Abrir resumen"

    - **NoSQL** abandona tablas/filas/SQL; las bases **documentales** (MongoDB) son una de varias familias (clave-valor, columnares, de grafos).
    - Un **documento** es un objeto autocontenido; una **colección** los agrupa sin esquema fijo — nada de tablas ni `JOIN`.
    - MongoDB usa BSON y `ObjectId` (alfanumérico) como identificador — distinto del `Long` autoincremental de JPA.
    - `libroId` en `Resena` es una relación **lógica**, no una clave foránea real — no hay integridad referencial automática entre dos motores distintos.
    - El patrón de **integridad referencial manual**: el código comprueba en PostgreSQL antes de tocar Mongo, porque el motor no puede hacerlo por sí solo.
    - La propiedad real de conexión es `spring.mongodb.uri` (no `spring.data.mongodb.uri`).
