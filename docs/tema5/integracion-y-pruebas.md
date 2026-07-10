<a id="integracion-y-pruebas"></a>

# 🧩 3. Integración y pruebas — cierre de RA6

!!! warning "🚧 Contenido pendiente de desarrollo"
    Esta página todavía no tiene la teoría redactada. Usa el prompt de más abajo con
    `/improve-notes`, apoyándote en el proyecto **GameVault** adjunto, para generar el
    contenido definitivo. Este apartado cierra el RA6 y todo el módulo.

---

## Prompt para `/improve-notes`

```text
Redacta el apartado de teoría "Integración y pruebas — cierre de RA6" del Tema 5 (RA6)
del módulo Acceso a Datos (0486), semana real 19 del calendario — apartado que CIERRA el
RA6 y todo el módulo 0486. Sigue las convenciones de estilo del README.md del repo.

Criterios de evaluación de RA6 que cubre este apartado (curriculum.md):
- h) Se han probado y documentado los componentes desarrollados.
- i) Se han integrado los componentes desarrollados en aplicaciones.

Contenido central: cómo se prueba y documenta un componente ya diseñado (repaso de
Testcontainers, ya visto en el Tema 3 para JSONB, ahora generalizado a todos los
componentes del proyecto), y cómo se demuestra que varios componentes, diseñados por
separado, funcionan correctamente cuando se integran juntos en la aplicación completa.

Apóyate en el proyecto GameVault (com.aleroig.gamevault) como ejemplo real y de cierre:
- src/test/java/com/aleroig/gamevault/integration/GamevaultApiTest.java: el test de
  integración más completo del proyecto, con `@Testcontainers` levantando
  simultáneamente PostgreSQL, MongoDB y RabbitMQ (`PostgreSQLContainer`,
  `MongoDBContainer`, `RabbitMQContainer`, más un contenedor Redis genérico) — preséntalo
  como el ejemplo definitivo de "probar la integración de componentes reales, no
  mockeados", que es exactamente lo que pide el criterio i) de este RA.
- src/test/java/com/aleroig/gamevault/catalogo/CatalogoConsultaServiceImplTest.java:
  ejemplo de test de un componente aislado (el creado en la Actividad 5.1), en contraste
  con el test de integración completo — explica cuándo conviene cada tipo de test
  (unitario/aislado para lógica interna de un componente, de integración para verificar
  que los componentes trabajan bien juntos).
- .github/workflows/ci.yml del proyecto: menciona brevemente que estos tests se ejecutan
  automáticamente en cada cambio (integración continua, ya visto conceptualmente en el
  Tema 0 con GitHub Actions) — cierra el círculo entre lo primero que se vio en el módulo
  (Git/GitHub/CI en el Tema 0) y lo último (probar componentes de acceso a datos).

Cierra el apartado (y el módulo completo) con una recapitulación de todo el recorrido del
curso: RA2 (conectores) → RA3 (ORM) → RA4 (JSONB/objeto-relacional) → RA5 (MongoDB/
documental) → RA6 (componentes que envuelven y desacoplan todo lo anterior), enlazando
con la Actividad 5.3, que es la entrega final integradora del módulo.
```
