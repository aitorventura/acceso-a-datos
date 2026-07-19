<a id="integracion-y-pruebas"></a>

# 🧩 3. Integración y pruebas

Último apartado del módulo. Responde a la pregunta que da sentido a todo lo anterior: ¿cómo sabes que tus componentes, diseñados por separado, funcionan bien **juntos**?

---

## 🧪 Probar un componente aislado — repaso

Ya construiste este tipo de test en la Actividad 5.1:

```java
@ExtendWith(MockitoExtension.class)
class CatalogoConsultaServiceImplTest {
    @Mock
    private VideojuegoRepository videojuegoRepository;

    @InjectMocks
    private CatalogoConsultaServiceImpl catalogoConsultaService;

    @Test
    void existeVideojuego_DebeDevolverTrue_CuandoExiste() {
        when(videojuegoRepository.existsById(1L)).thenReturn(true);
        assertTrue(catalogoConsultaService.existeVideojuego(1L));
    }
}
```

Un test aislado con mocks prueba la **lógica interna** de un componente, sin ninguna base de datos real de por medio — rapidísimo, perfecto para verificar el comportamiento de una pieza concreta.

---

## 🔗 Probar la integración de todo junto

El test de integración más completo no mockea nada — levanta **todos** los motores reales que usa la aplicación, simultáneamente, en contenedores Docker, solo para la duración del test. En el caso de la librería, sus dos motores:

```java
@Testcontainers
class LibreriaApiTest {

    @Container
    static PostgreSQLContainer<?> postgres = new PostgreSQLContainer<>("postgres:16-alpine");

    @Container
    static MongoDBContainer mongodb = new MongoDBContainer("mongo:7");

    // ...
}
```

PostgreSQL y MongoDB — los servicios reales del `docker-compose.yaml`, arrancados solo para este test (si la aplicación usara más servicios, como una cola de mensajes o una caché, se añadirían aquí exactamente igual, un `@Container` por servicio). Este es el ejemplo definitivo de "probar la integración de componentes reales, no mockeados": que los componentes desarrollados por separado (el catálogo en PostgreSQL, las reseñas en MongoDB) funcionen correctamente cuando se integran juntos, en la aplicación completa.

### Cuándo conviene cada tipo de test

| | Test aislado (mocks) | Test de integración (Testcontainers) |
|---|---|---|
| Qué prueba | La lógica interna de un componente | Que varios componentes reales trabajan bien juntos |
| Velocidad | Muy rápido | Más lento (levanta contenedores reales) |
| Cuándo usarlo | Casos de la lógica de un componente concreto (validaciones, cálculos) | Flujos completos, entre módulos, entre motores distintos |

Ninguno sustituye al otro — un proyecto real necesita ambos niveles.

---

## 📝 "Documentar" un componente

Documentar un componente no es sobre todo escribir un documento externo aparte — es que el propio test, con nombres descriptivos (`existeLibro_DebeDevolverTrue_CuandoExiste`) y una estructura clara, deje constancia de qué comportamiento se espera y en qué condiciones. El test bien escrito **es** la documentación viva del componente — se actualiza sola cuando el comportamiento cambia (si no, el test falla y te avisa).

---

## 🔄 El círculo se cierra: CI

`.github/workflows/ci.yml`, que configuraste en el Tema 0 con GitHub Actions, ejecuta estos mismos tests automáticamente en cada cambio que subes al repositorio — sin que tengas que acordarte de ejecutarlos tú a mano cada vez. Es el mismo concepto de integración continua que viste al principio del módulo, cerrando el círculo con lo último que has construido: probar componentes de acceso a datos.

---

## ✅ Ideas clave

??? tip "Abrir resumen"

    - Un test **aislado** (mocks) prueba la lógica interna de un componente; un test de **integración** (Testcontainers) prueba que varios componentes reales funcionan bien juntos.
    - Un test de integración completo levanta todos los motores reales de la aplicación (PostgreSQL, MongoDB...) simultáneamente en Docker, solo para el test.
    - Ninguno de los dos niveles de test sustituye al otro — un proyecto real necesita ambos.
    - "Documentar" un componente es, sobre todo, que el propio test (nombre + estructura) deje claro qué se espera de él.
    - El CI configurado en el Tema 0 ejecuta estos tests automáticamente en cada cambio — cerrando el círculo entre el principio y el final del módulo.
