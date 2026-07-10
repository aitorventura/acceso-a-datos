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
{ "_id": ObjectId("..."), "titulo": "Hades", "puntuacion": 9 }

// El equivalente a un SELECT sencillo
db.videojuegos.find({ "titulo": "Hades" })
```

Vale la pena verlo en esta forma cruda antes de que Spring lo esconda tras un repositorio.

---

## ⚖️ Cuándo encaja lo documental (intuición)

Datos autocontenidos, de estructura variable de un registro a otro: bien. Relaciones fuertes entre entidades y transacciones complejas que las involucren: mejor relacional. Profundizarás en esta comparación en el siguiente apartado — de momento, quédate con la intuición.

---

## 🎮 Aterrizaje en GameVault: el módulo `reviews`

### Por qué dos bases de datos distintas

GameVault usa PostgreSQL para el catálogo (`Videojuego`/`Estudio`: relaciones claras, necesidad de transacciones ACID fuertes) y **MongoDB** para las reseñas (`Review`: documentos independientes entre sí, sin relaciones que mantener, con forma que podría evolucionar con el tiempo). Es una decisión de arquitectura, no una moda — cada motor se usa donde encaja mejor.

### La entidad `Review`

```java
@Document(collection = "review")
public class Review {

    @Id
    private String id; // alfanumérico, tipo ObjectId — no un Long autogenerado

    private Long videojuegoId; // relación "lógica" con el catálogo en PostgreSQL
    private String autor;
    private Integer puntuacion;
    private String comentario;
}
```

`@Document(collection = "review")` es el equivalente Mongo de `@Entity`/`@Table` — declara en qué colección vive. El `@Id` es `String`, no `Long`: en Mongo los identificadores son alfanuméricos por naturaleza (`ObjectId`), a diferencia del autoincremental que has usado siempre en PostgreSQL.

!!! warning "`videojuegoId` no es una clave foránea real"
    Aunque el campo se llame `videojuegoId` y apunte conceptualmente a un `Videojuego` de PostgreSQL, **no hay ninguna integridad referencial automática** entre dos motores de base de datos distintos. MongoDB no sabe nada de PostgreSQL, ni al revés — es responsabilidad de tu código mantener esa relación con sentido.

### El repositorio, con naming de método (igual que en JPA)

```java
public interface ReviewRepository extends MongoRepository<Review, String> {
    List<Review> findByVideojuegoId(Long videojuegoId);
}
```

Spring Data también genera consultas automáticas por el nombre del método en MongoDB, exactamente igual que hacía `JpaRepository` en el Tema 2 — sin que escribas una sola query explícita.

### El patrón de "integridad referencial manual"

```java
public List<ReviewResponseDTO> findByVideojuegoId(Long videojuegoId) {
    // 1. Verificamos que el juego exista en PostgreSQL antes de buscar sus reseñas
    if (!catalogoConsultaService.existeVideojuego(videojuegoId)) {
        throw new ResponseStatusException(HttpStatus.NOT_FOUND, "Videojuego no encontrado en el catálogo");
    }
    // 2. Buscamos en MongoDB y mapeamos
    return reviewRepository.findByVideojuegoId(videojuegoId).stream().map(this::mapToDTO).toList();
}
```

Este es el patrón central de este apartado: como Mongo no puede validar una clave foránea hacia una tabla de PostgreSQL, el propio código de la aplicación tiene que hacerlo — consultando primero `CatalogoConsultaService.existeVideojuego(...)` (el mismo componente del paquete `catalogo.api` que viste en el Tema 2) antes de tocar Mongo.

### Una agregación sencilla, en memoria

```java
public ReviewResumenDTO getResumenByVideojuegoId(Long videojuegoId) {
    List<Review> reviews = reviewRepository.findByVideojuegoId(videojuegoId);
    long totalReviews = reviews.size();
    double puntuacionMedia = reviews.stream().mapToInt(Review::getPuntuacion).average().orElse(0.0);
    return new ReviewResumenDTO(videojuegoId, totalReviews, puntuacionMedia);
}
```

`getResumenByVideojuegoId` calcula el total y la media de puntuación **en memoria**, con streams de Java, tras traer todos los documentos de Mongo — no usa el framework de agregación nativo de Mongo (`$group`, `$avg`). Es una simplificación deliberada: funciona bien con pocos documentos por videojuego, y queda como posible mejora si algún día hubiera muchísimas reseñas por consultar.

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
    uri: mongodb://localhost:27017/gamevault_db
```

!!! warning "La propiedad real es `spring.mongodb.uri`, no `spring.data.mongodb.uri`"
    Es fácil confundirse con tutoriales antiguos: en este proyecto, `mongodb` cuelga directamente de `spring`, no de `spring.data`. Comprueba siempre el `application-dev.yaml` real antes de dar por buena una propiedad.

En paralelo al `datasource` de PostgreSQL que ya conoces del Tema 1, esta es toda la configuración necesaria para conectar con MongoDB.

---

## ✅ Ideas clave

??? tip "Abrir resumen"

    - **NoSQL** abandona tablas/filas/SQL; las bases **documentales** (MongoDB) son una de varias familias (clave-valor, columnares, de grafos).
    - Un **documento** es un objeto autocontenido; una **colección** los agrupa sin esquema fijo — nada de tablas ni `JOIN`.
    - MongoDB usa BSON y `ObjectId` (alfanumérico) como identificador — distinto del `Long` autoincremental de JPA.
    - `videojuegoId` en `Review` es una relación **lógica**, no una clave foránea real — no hay integridad referencial automática entre dos motores distintos.
    - El patrón de **integridad referencial manual**: el código comprueba en PostgreSQL antes de tocar Mongo, porque el motor no puede hacerlo por sí solo.
    - La propiedad real de conexión es `spring.mongodb.uri` (no `spring.data.mongodb.uri`).
