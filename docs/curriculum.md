# Currículo: Módulo 0486 — Acceso a Datos

**Equivalencia ECTS:** 9  
**Duración:** 80 horas

---

## Resultados de Aprendizaje y Criterios de Evaluación

### RA1 — Gestión de Ficheros
> Desarrolla aplicaciones que gestionan información almacenada en ficheros identificando el campo de aplicación de los mismos y utilizando clases específicas.

- a) Se han utilizado clases para la gestión de ficheros y directorios.
- b) Se han valorado las ventajas y los inconvenientes de las distintas formas de acceso.
- c) Se han utilizado clases para recuperar información almacenada en ficheros.
- d) Se han utilizado clases para almacenar información en ficheros.
- e) Se han utilizado clases para realizar conversiones entre diferentes formatos de ficheros.
- f) Se han previsto y gestionado las excepciones.
- g) Se han probado y documentado las aplicaciones desarrolladas.

### RA2 — Acceso a Bases de Datos Relacionales
> Desarrolla aplicaciones que gestionan información almacenada en bases de datos relacionales identificando y utilizando mecanismos de conexión.

- a) Se han valorado las ventajas e inconvenientes de utilizar conectores.
- b) Se han utilizado gestores de bases de datos embebidos e independientes.
- c) Se ha utilizado el conector idóneo en la aplicación.
- d) Se ha establecido la conexión.
- e) Se ha definido la estructura de la base de datos.
- f) Se han desarrollado aplicaciones que modifican el contenido de la base de datos.
- g) Se han definido los objetos destinados a almacenar el resultado de las consultas.
- h) Se han desarrollado aplicaciones que efectúan consultas.
- i) Se han eliminado los objetos una vez finalizada su función.
- j) Se han gestionado las transacciones.
- k) Se han ejecutado procedimientos almacenados en la base de datos.

### RA3 — Persistencia mediante Herramientas ORM
> Gestiona la persistencia de los datos identificando herramientas de mapeo objeto relacional (ORM) y desarrollando aplicaciones que las utilizan.

- a) Se ha instalado la herramienta ORM.
- b) Se ha configurado la herramienta ORM.
- c) Se han definido configuraciones de mapeo.
- d) Se han aplicado mecanismos de persistencia a los objetos.
- e) Se han desarrollado aplicaciones que modifican y recuperan objetos persistentes.
- f) Se han desarrollado aplicaciones que realizan consultas usando el lenguaje SQL.
- g) Se han gestionado las transacciones.

### RA4 — Bases de Datos Objeto Relacionales y Orientadas a Objetos
> Desarrolla aplicaciones que gestionan la información almacenada en bases de datos objeto relacionales y orientadas a objetos valorando sus características y utilizando los mecanismos de acceso incorporados.

- a) Se han identificado las ventajas e inconvenientes de las bases de datos que almacenan objetos.
- b) Se han establecido y cerrado conexiones.
- c) Se ha gestionado la persistencia de objetos simples.
- d) Se ha gestionado la persistencia de objetos estructurados.
- e) Se han desarrollado aplicaciones que realizan consultas.
- f) Se han modificado los objetos almacenados.
- g) Se han gestionado las transacciones.
- h) Se han probado y documentado las aplicaciones desarrolladas.

### RA5 — Bases de Datos Documentales Nativas
> Desarrolla aplicaciones que gestionan la información almacenada en bases de datos documentales nativas evaluando y utilizando clases específicas.

- a) Se han valorado las ventajas e inconvenientes de utilizar bases de datos documentales nativas.
- b) Se ha establecido la conexión con la base de datos.
- c) Se han desarrollado aplicaciones que efectúan consultas sobre el contenido de la base de datos.
- d) Se han añadido y eliminado colecciones de la base de datos.
- e) Se han desarrollado aplicaciones para añadir, modificar y eliminar documentos de la base de datos.

### RA6 — Componentes de Acceso a Datos
> Programa componentes de acceso a datos identificando las características que debe poseer un componente y utilizando herramientas de desarrollo.

- a) Se han valorado las ventajas e inconvenientes de utilizar programación orientada a componentes.
- b) Se han identificado herramientas de desarrollo de componentes.
- c) Se han programado componentes que gestionan información almacenada en ficheros.
- d) Se han programado componentes que gestionan mediante conectores información almacenada en bases de datos.
- e) Se han programado componentes que gestionan información usando mapeo objeto relacional.
- f) Se han programado componentes que gestionan información almacenada en bases de datos objeto relacionales y orientadas a objetos.
- g) Se han programado componentes que gestionan información almacenada en una base de datos documental nativa.
- h) Se han probado y documentado los componentes desarrollados.
- i) Se han integrado los componentes desarrollados en aplicaciones.

---

## Contenidos Básicos

### Manejo de ficheros (→ RA1)
- Clases asociadas a las operaciones de gestión de ficheros y directorios: creación, borrado, copia, movimiento, recorrido, entre otras.
- Formas de acceso a un fichero. Ventajas.
- Clases para gestión de flujos de datos desde/hacia ficheros. Flujos de bytes y de caracteres.
- Operaciones sobre ficheros secuenciales y aleatorios.
- Serialización/deserialización de objetos.
- Trabajo con ficheros de intercambio de datos (XML y JSON, entre otros). Analizadores sintácticos (parser) y vinculación (binding). Conversión entre formatos.
- Excepciones: detección y tratamiento.
- Desarrollo de aplicaciones que utilizan ficheros.

### Manejo de conectores (→ RA2)
- El desfase objeto-relacional.
- Protocolos de acceso a bases de datos. Conectores.
- Establecimiento de conexiones. Pooling de conexiones.
- Ejecución de sentencias de descripción de datos.
- Ejecución de sentencias de modificación de datos.
- Ejecución de consultas. Manipulación del resultado.
- Ejecución de procedimientos almacenados en la base de datos. Parámetros.
- Gestión de transacciones.
- Desarrollo de programas que utilizan bases de datos.

### Herramientas de mapeo objeto relacional (ORM) (→ RA3)
- Concepto de mapeo objeto relacional.
- Características de las herramientas ORM. Herramientas ORM más utilizadas.
- Instalación y configuración de una herramienta ORM.
- Estructura de un fichero de mapeo. Elementos, propiedades.
- Mapeo basado en anotaciones.
- Clases persistentes. Sesiones; estados de un objeto.
- Carga, almacenamiento y modificación de objetos.
- Consultas SQL.
- Gestión de transacciones.
- Desarrollo de programas que utilizan bases de datos a través de herramientas ORM.

### Bases de datos objeto relacionales y orientadas a objetos (→ RA4)
- Gestores de bases de datos objeto relacionales. Características. Ventajas.
- Gestión de objetos con SQL; ANSI SQL.
- Acceso a las funciones del gestor de base de datos objeto-relacional desde el lenguaje de programación.
- Gestores de bases de datos orientadas a objetos. Características. Ventajas.
- Gestión de la persistencia de objetos.
- El interfaz de programación de aplicaciones de la base de datos orientada a objetos. Consultas y persistencia de datos. Lenguaje OQL.
- Gestión de transacciones.
- Desarrollo de programas que gestionan objetos en bases de datos.

### Bases de datos documentales (→ RA5)
- Bases de datos documentales nativas. Características. Ventajas.
- Establecimiento y cierre de conexiones.
- Colecciones y documentos.
- Creación y borrado de colecciones; clases y métodos.
- Añadir, modificar y eliminar documentos; clases y métodos.
- Lenguajes de consulta. Realización de consultas; clases y métodos.
- Desarrollo de programas que utilizan bases de datos documentales.

### Programación de componentes de acceso a datos (→ RA6)
- Concepto de componente; características. Ventajas.
- Propiedades y atributos.
- Eventos; asociación de acciones a eventos.
- Persistencia del componente. Serialización.
- Herramientas para desarrollo de componentes.
- Desarrollo, empaquetado y utilización de componentes.