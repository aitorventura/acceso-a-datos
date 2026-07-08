# Actividad 0.5: Pipeline de CI para un proyecto Java real

!!! warning "Descarga la plantilla"
    📄 [Plantilla 0.5 — Pipeline de CI para un proyecto Java real](plantillas/Actividad_0_5_Plantilla.docx){target="_blank" rel="noopener"}

!!! warning "Antes de empezar"
    Lee la actividad entera antes de ejecutar nada.

---

## ¿Qué vas a practicar?

En esta actividad vas a montar tu propio pipeline de CI con un proyecto nuevo: un **validador de DNI**, una utilidad con lógica real (cálculo de letra de control) que cubre todos los elementos vistos en la teoría: tests, artefactos, jobs encadenados y secrets.

## Requisitos previos

- Cuenta de GitHub activa y acceso a internet.
- Java 17+ y Maven instalados en tu equipo (para poder probar el proyecto en local antes de subirlo).
- No hace falta nada más: el resto se ejecuta en los servidores de GitHub.

---

## Parte A — Un proyecto real con CI desde el primer commit

**Paso 1.** Crea un repositorio nuevo en GitHub llamado `validador-dni-ci`. Márcalo como público. No lo inicialices con README todavía: lo subirás tú con el resto del proyecto.

**Paso 2.** En tu máquina, crea la siguiente estructura de carpetas y ficheros:

```
validador-dni-ci/
├── .github/
│   └── workflows/
│       └── ci.yml
├── src/
│   ├── main/java/com/ejemplo/
│   │   ├── ValidadorDni.java
│   │   └── ValidadorDniApp.java
│   └── test/java/com/ejemplo/
│       └── ValidadorDniTest.java
├── pom.xml
└── README.md
```

**`pom.xml`**

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0">
  <modelVersion>4.0.0</modelVersion>
  <groupId>com.ejemplo</groupId>
  <artifactId>validador-dni</artifactId>
  <version>1.0.0</version>
  <packaging>jar</packaging>

  <properties>
    <maven.compiler.source>17</maven.compiler.source>
    <maven.compiler.target>17</maven.compiler.target>
    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
  </properties>

  <dependencies>
    <dependency>
      <groupId>org.junit.jupiter</groupId>
      <artifactId>junit-jupiter</artifactId>
      <version>5.10.2</version>
      <scope>test</scope>
    </dependency>
  </dependencies>

  <build>
    <finalName>validador-dni</finalName>
    <plugins>
      <plugin>
        <groupId>org.apache.maven.plugins</groupId>
        <artifactId>maven-surefire-plugin</artifactId>
        <version>3.2.5</version>
      </plugin>
      <plugin>
        <groupId>org.apache.maven.plugins</groupId>
        <artifactId>maven-jar-plugin</artifactId>
        <version>3.4.1</version>
        <configuration>
          <archive>
            <manifest>
              <mainClass>com.ejemplo.ValidadorDniApp</mainClass>
            </manifest>
          </archive>
        </configuration>
      </plugin>
    </plugins>
  </build>
</project>
```

**`src/main/java/com/ejemplo/ValidadorDni.java`**

```java
package com.ejemplo;

public class ValidadorDni {

    private static final String LETRAS = "TRWAGMYFPDXBNJZSQVHLCKE";

    public boolean esValido(String dni) {
        if (dni == null) {
            return false;
        }
        String limpio = dni.trim().toUpperCase();
        if (!limpio.matches("\\d{8}[A-Z]")) {
            return false;
        }
        String numero = limpio.substring(0, 8);
        char letraEsperada = calcularLetra(numero);
        char letraDada = limpio.charAt(8);
        return letraEsperada == letraDada;
    }

    public char calcularLetra(String numero) {
        int valor = Integer.parseInt(numero);
        return LETRAS.charAt(valor % 23);
    }
}
```

**`src/main/java/com/ejemplo/ValidadorDniApp.java`**

```java
package com.ejemplo;

public class ValidadorDniApp {
    public static void main(String[] args) {
        if (args.length != 1) {
            System.out.println("Uso: java -jar validador-dni.jar <DNI>");
            return;
        }
        ValidadorDni validador = new ValidadorDni();
        boolean valido = validador.esValido(args[0]);
        System.out.println(args[0] + " -> " + (valido ? "VÁLIDO" : "NO VÁLIDO"));
    }
}
```

**`src/test/java/com/ejemplo/ValidadorDniTest.java`**

```java
package com.ejemplo;

import org.junit.jupiter.api.Test;
import static org.junit.jupiter.api.Assertions.*;

class ValidadorDniTest {

    private final ValidadorDni validador = new ValidadorDni();

    @Test
    void dniValidoConLetraCorrecta() {
        assertTrue(validador.esValido("12345678Z"));
    }

    @Test
    void dniConLetraIncorrecta() {
        assertFalse(validador.esValido("12345678A"));
    }

    @Test
    void dniAceptaLetraMinuscula() {
        assertTrue(validador.esValido("12345678z"));
    }

    @Test
    void dniConLongitudIncorrecta() {
        assertFalse(validador.esValido("1234567Z"));
    }

    @Test
    void calcularLetraDevuelveLaCorrecta() {
        assertEquals('Z', validador.calcularLetra("12345678"));
    }
}
```

**Paso 3.** Crea `.github/workflows/ci.yml` con un workflow que compile y teste el proyecto en cada push:

```yaml
name: CI — Validador DNI

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - name: Descargar el código fuente
        uses: actions/checkout@v4

      - name: Instalar Java 17
        uses: actions/setup-java@v4
        with:
          java-version: '17'
          distribution: 'temurin'
          cache: 'maven'

      - name: Ejecutar los tests
        run: mvn test -B

      - name: Guardar el informe de tests
        if: always()
        uses: actions/upload-artifact@v4
        with:
          name: informe-tests
          path: target/surefire-reports/
```

!!! warning "Predicción antes de hacer push"
    Antes de subir el proyecto, responde por escrito: ¿cuántos tests se van a ejecutar? ¿Qué resultado esperas ver en el log? ¿Qué pasaría si te falta el `pom.xml`?

**Paso 4.** Inicializa el repositorio Git en local, haz commit de todo y haz push a GitHub:

```bash
git init
git add .
git commit -m "Proyecto validador de DNI con CI"
git branch -M main
git remote add origin <URL_DE_TU_REPO>
git push -u origin main
```

**Paso 5.** Ve a la pestaña **Actions** y observa el workflow ejecutarse.

**Pregunta A.1.** ¿Ha coincidido tu predicción? Copia del log la línea exacta que indica cuántos tests se han ejecutado y cuántos han fallado.

**Pregunta A.2.** El step "Ejecutar los tests" usa `mvn test`, no `mvn package` ni `mvn install`. ¿Qué diferencia hay entre esos tres comandos de Maven? ¿Por qué basta con `mvn test` para este job?

---

## Parte B — Empaquetar y ejecutar un artefacto real

Comprobar que los tests pasan está bien, pero un pipeline de CI de verdad también produce algo entregable: el `.jar` ejecutable. Vas a añadir un segundo job que solo se ejecuta si los tests han pasado, y que empaqueta el proyecto en un `.jar` descargable y ejecutable por línea de comandos.

**Paso 6.** Modifica `ci.yml` para añadir el job `empaquetar`:

```yaml
name: CI — Validador DNI

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - name: Descargar el código fuente
        uses: actions/checkout@v4

      - name: Instalar Java 17
        uses: actions/setup-java@v4
        with:
          java-version: '17'
          distribution: 'temurin'
          cache: 'maven'

      - name: Ejecutar los tests
        run: mvn test -B

      - name: Guardar el informe de tests
        if: always()
        uses: actions/upload-artifact@v4
        with:
          name: informe-tests
          path: target/surefire-reports/

  empaquetar:
    runs-on: ubuntu-latest
    needs: test
    steps:
      - name: Descargar el código fuente
        uses: actions/checkout@v4

      - name: Instalar Java 17
        uses: actions/setup-java@v4
        with:
          java-version: '17'
          distribution: 'temurin'
          cache: 'maven'

      - name: Empaquetar el JAR
        run: mvn package -DskipTests -B

      - name: Guardar el JAR como artefacto
        uses: actions/upload-artifact@v4
        with:
          name: validador-dni-jar
          path: target/validador-dni.jar
```

!!! warning "Predicción antes de hacer push"
    ¿Por qué crees que el job `empaquetar` usa `-DskipTests`, si los tests ya se han ejecutado en el job `test`? ¿Qué pasaría si no estuviera `needs: test`?

**Paso 7.** Haz push y observa los dos jobs en el panel Actions.

**Paso 8.** Cuando el workflow termine en verde, entra al run y descarga el artefacto `validador-dni-jar` desde la parte inferior de la página del run.

**Paso 9.** En tu máquina, ejecuta el `.jar` descargado con un DNI real (el tuyo, calculando la letra con cualquier validador online, o uno inventado con letra correcta):

```bash
java -jar validador-dni.jar 12345678Z
```

**Pregunta B.1.** ¿Para qué sirve `needs: test` aquí? Si lo quitaras, ¿podría llegar a publicarse un JAR de una versión que no pasa los tests?

**Pregunta B.2.** ¿Por qué tiene sentido usar `-DskipTests` en el job `empaquetar` en vez de repetir `mvn test` otra vez?

---

## Parte C — Un fallo real, no un comando inventado

Vas a romper el código de verdad, como ocurriría si alguien introduce un bug sin darse cuenta.

**Paso 10.** En `ValidadorDni.java`, quita el `.toUpperCase()` de la línea que limpia el DNI:

```java
String limpio = dni.trim();   // bug: ya no convierte a mayúsculas
```

!!! warning "Predicción antes de hacer push"
    ¿Qué test crees que va a fallar con este cambio? Antes de mirar el código del test, intenta predecirlo tú mismo: ¿qué tiene de especial ese test frente a los demás?

**Paso 11.** Haz commit y push. Observa qué ocurre con los dos jobs (`test` y `empaquetar`).

**Pregunta C.1.** ¿Ha coincidido tu predicción? Copia la línea exacta del log que muestra el test que ha fallado y el motivo.

**Pregunta C.2.** ¿Se ha llegado a ejecutar el job `empaquetar`? ¿Por qué es importante que no se genere un JAR cuando los tests fallan?

**Paso 12.** Corrige el bug (vuelve a poner `.trim().toUpperCase()`), haz commit y push. Comprueba que el pipeline completo vuelve a verde y que el JAR vuelve a estar disponible como artefacto.

---

## Parte D — Un secret que genera algo real

Vas a usar un secret para generar un fichero de metadatos de la build que incluya un identificador de despliegue, sin que ese identificador aparezca nunca en el log.

**Paso 13.** En tu repositorio, ve a **Settings → Secrets and variables → Actions → New repository secret**. Crea un secret llamado `DEPLOY_ID` con cualquier valor que inventes (por ejemplo `equipo-7-prod-2024`).

**Paso 14.** Añade tú mismo un tercer job llamado `generar_manifiesto` a `ci.yml`. El job tiene que cumplir estas condiciones — cómo lograrlo es parte del reto:

- Solo se ejecuta si el job `empaquetar` ha terminado correctamente.
- Genera un fichero `build-info.txt` que contenga el nombre del proyecto, el hash del commit (`github.sha`), la fecha y hora UTC, y un hash de 12 caracteres derivado del secret `DEPLOY_ID` (el valor del secret no debe aparecer en texto plano en ningún sitio).
- Sube ese fichero como artefacto llamado `build-info`.

!!! tip "Por dónde investigar"
    Puedes consultar los apartados 5.3 y 5.8 de la teoría, la documentación oficial de GitHub Actions en [docs.github.com/actions](https://docs.github.com/en/actions) y los ejemplos de los jobs anteriores de este mismo workflow.

    Para la línea del hash, esta es la pista clave — el resto del job lo escribes tú:

    ```bash
    echo "Deploy ID (hash): $(echo -n "$DEPLOY_ID" | sha256sum | cut -c1-12)" >> build-info.txt
    ```

**Paso 15.** Haz push, espera a que termine el workflow y descarga el artefacto `build-info`. Ábrelo y comprueba su contenido.

**Pregunta D.1.** ¿Aparece el valor real de `DEPLOY_ID` en algún punto del log de este job? Compruébalo expandiendo el step y busca específicamente la línea del `echo`.

**Pregunta D.2.** ¿Qué utilidad real tendría un fichero como `build-info.txt` en un proyecto en producción? Piensa en quién lo consultaría y para qué.

**Pregunta D.3.** Si dos ejecuciones del workflow generan manifiestos con el mismo hash de `DEPLOY_ID`, ¿qué te indica eso sobre el secret usado en ambas? ¿Y si el hash cambiara de una ejecución a otra sin que tú hayas tocado el secret?

---

## Parte E — Tabla de eventos y decisiones

Sin tocar el repositorio, responde en la tabla siguiente qué evento usarías en cada situación. Justifica cada elección con una frase.

| Situación | Evento que usarías | Por qué |
|---|---|---|
| Ejecutar el pipeline completo cada vez que alguien sube código a cualquier rama | | |
| Ejecutar solo los tests cuando se propone un merge a `main` | | |
| Generar y publicar el JAR automáticamente cuando se crea un tag `v1.0` | | |
| Regenerar el manifiesto de build todas las noches a las 3:00 aunque no haya cambios | | |

!!! tip "Pista"
    Los eventos más habituales son: `push`, `pull_request`, `release` y `schedule` (con expresión cron). Puedes consultarlos en el apartado 5.2 de la teoría.

---

## Solución

!!! tip "Descarga la solución"
    📄 [Solución 0.5 — Pipeline de CI para un proyecto Java real](plantillas/Actividad_0_5_Solucion.docx){target="_blank" rel="noopener"}

    Esta actividad es de repaso/introducción y no se entrega: practica los pasos por tu cuenta y usa la solución para autoevaluarte al terminar (o si te atascas en algún paso).
