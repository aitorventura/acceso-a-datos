<a id="spring-boot-y-gamevault"></a>

# 🧩 1. Spring Boot: el chasis de GameVault

!!! warning "🚧 Contenido pendiente de desarrollo"
    Esta página todavía no tiene la teoría redactada. Usa el prompt de más abajo con
    `/improve-notes`, apoyándote en el proyecto **GameVault** adjunto, para generar el
    contenido definitivo.

---

## Prompt para `/improve-notes`

```text
Redacta el apartado de teoría "Spring Boot: el chasis de GameVault" del Tema 1 (RA2 -
Manejo de conectores) del módulo Acceso a Datos (0486), semana real 3 del calendario.
Sigue las convenciones de estilo del README.md del repo.

PROPÓSITO: este apartado es la BASE de todo el curso y no corresponde a ningún criterio
de evaluación concreto — existe porque el alumnado NO conoce Spring y sin esto no puede
entender nada de lo que viene (ni en AD ni en PSP, que comparten el proyecto GameVault).
El alumnado llega sabiendo: Java (POO, colecciones), Git/GitHub y Docker (Tema 0), y
bases de datos relacionales/SQL de 1º de DAM. NO sabe qué es un framework, ni Maven, ni
qué es una aplicación web. Explica TODO desde cero, sin dar nada por conocido.

Estructura del apartado — de lo general a GameVault:

PARTE 1 — Conceptos desde cero:
- Qué es una aplicación de servidor (backend): un programa Java que no tiene ventanas ni
  consola interactiva — arranca, se queda escuchando, y responde a peticiones de otros
  programas. Contrasta con las aplicaciones de consola que el alumnado ha hecho hasta
  ahora.
- Qué es un framework y por qué existe: la diferencia entre "yo llamo a una librería" y
  "el framework llama a mi código" (inversión de control), con una analogía sencilla.
- Qué es Spring y qué añade Spring Boot (configuración automática, servidor embebido,
  starters) — 3-4 frases por concepto, sin historia innecesaria.
- Qué es Maven y el pom.xml: el gestor de dependencias — qué problema resuelve
  (descargar y versionar librerías) y cómo se lee una dependencia en el pom.
- La inyección de dependencias explicada desde cero con un ejemplo mínimo genérico (una
  clase que necesita otra: versión "new a mano" vs. versión "Spring me la da"), y las
  anotaciones que marcan qué gestiona Spring: @Service, @Repository, @RestController,
  @Configuration — qué tienen en común (todas declaran "esto es un bean").

PARTE 2 — GameVault por dentro (el aterrizaje):
- El main de GamevaultApplication.java y @SpringBootApplication: qué pasa al arrancar.
- El pom.xml real: localizar 2-3 starters y leer qué aporta cada uno — usa los
  artifactId REALES del proyecto (`spring-boot-starter-webmvc` y
  `spring-boot-starter-data-jpa`; NO `spring-boot-starter-web`, que es el nombre
  genérico de versiones antiguas de Spring Boot — este proyecto usa Spring Boot 4, que
  separó el starter web en `webmvc`/`webflux`). Es el mismo starter que el alumnado ya
  eligió al generar su propio proyecto en la Actividad 1.1.
- La estructura de paquetes real (catalogo, reviews, actividad, seguridad, config,
  exception) y la arquitectura en capas dentro de cada módulo: controller (recibe
  peticiones) → service (lógica) → repository (datos), usando VideojuegoController →
  VideojuegoService → VideojuegoRepository como ejemplo visible, sin entrar aún en qué
  hace cada anotación de persistencia (eso llega en los apartados siguientes). ATENCIÓN:
  deja explícito en el propio texto que esto es una lectura de la REFERENCIA del
  profesor, no del proyecto del alumnado — el alumnado solo tiene, en este punto del
  curso, el proyecto vacío recién creado en la Actividad 1.1 (o, como mucho, las
  entidades Videojuego/Estudio si esta página se lee después de esa actividad); su
  propio VideojuegoController/Service/Repository con el CRUD completo lo construirá la
  semana que viene, en la Actividad 1.2. No debe parecer que ya debería tenerlo.
- application.yaml / application-dev.yaml: dónde vive la configuración y qué es un
  perfil (dev), en 3-4 frases.
- El @RequiredArgsConstructor de Lombok que verán por todo el proyecto: qué genera y por
  qué el proyecto lo usa para inyectar dependencias (constructor + final).

Cierra situando al alumnado: "este chasis lo iremos rellenando durante todo el curso
desde dos módulos a la vez — en AD la persistencia (lo que hay del service hacia abajo)
y en PSP la parte de servicios en red y seguridad (del controller hacia fuera)". Este
apartado se estudia la misma semana en que PSP empieza a leer el controlador REST:
manténlo coordinado y no expliques aquí HTTP/REST en detalle (es de PSP).
```
