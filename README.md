# Despliegues continuos: fundamentos, pipelines y nube


> **Objetivo del curso.** Implementar un flujo de integración y entrega de una aplicación: desde un cambio versionado hasta una revisión operativa en la nube, con pruebas, trazabilidad, control de acceso y mecanismos de observación y recuperación.

## Contenido

1. [Ruta del syllabus y resultados](#ruta)
2. [Unidad 1. CI/CD y control de versiones](#unidad-1)
3. [Unidad 2. Automatización de pipelines](#unidad-2)
4. [Unidad 3. Despliegue en la nube](#unidad-3)
5. [Laboratorio integrador reproducible](#laboratorio)
6. [Diagnóstico de fallas y recuperación](#diagnostico)
7. [Actividades, preguntas y evaluación](#actividades)
8. [Glosario y fuentes](#fuentes)

<a id="ruta"></a>
## 1. Ruta del syllabus y resultados

| Unidad | Temas del syllabus | Acompañamiento directo | Trabajo independiente | Total | Resultado principal |
|---|---|---:|---:|---:|---|
| 1. Fundamentos de CI/CD y control de versiones | CI, CD, beneficios, Git, repositorios remotos, flujo básico | 16 h | 32 h | 48 h | Analizar principios y componentes |
| 2. Automatización de pipelines | Pipeline, GitHub Actions/GitLab CI/Jenkins, configuración, pruebas, variables, errores | 16 h | 32 h | 48 h | Configurar pipelines |
| 3. Despliegue de aplicaciones en la nube | Infraestructura cloud, servicios, aplicaciones web, automatización, seguridad, monitoreo | 32 h | 64 h | 96 h | Desplegar aplicaciones |
| **Suma de unidades** | | **64 h** | **128 h** | **192 h** | |

**Nota de consistencia:** los datos generales indican 192 horas y 64 + 128 = 192. La última fila de la tabla original indica 196: es una discrepancia aritmética del syllabus. También repite el número 5 en los dos últimos tópicos de la unidad 3. Aquí se emplea la suma verificable de 192 horas.

~~~mermaid
pie showData
    title Distribución de 192 horas por unidad
    "Unidad 1: fundamentos" : 48
    "Unidad 2: pipelines" : 48
    "Unidad 3: nube" : 96
~~~

~~~mermaid
pie showData
    title Modalidad de trabajo: 192 horas
    "Acompañamiento directo" : 64
    "Trabajo independiente" : 128
~~~

**Competencias observables.** Al terminar, el estudiante puede: (1) explicar qué valida cada puerta de calidad; (2) demostrar que un cambio en una rama dispara pruebas sin acceder a credenciales de producción; (3) desplegar desde la rama autorizada; (4) identificar commit, ejecución y revisión; (5) localizar una falla y recuperar el servicio. Los resultados proceden del syllabus; las evidencias propuestas son una operacionalización didáctica.

<a id="unidad-1"></a>
## 2. Unidad 1. CI/CD y control de versiones

### 2.1. El problema que resuelve

Si los cambios se integran tarde, aparecen conflictos, pruebas manuales difíciles de repetir y diferencias entre lo que se verificó y lo que se publicó. Un flujo CI/CD reduce la distancia entre cambio y retroalimentación mediante ejecución automática y criterios explícitos. No elimina los defectos: hace más visible cuándo y dónde falló una verificación. [GitHub Actions, F2](#ref-f2); [DORA, F12](#ref-f12).

| Término | Qué ocurre | Evidencia mínima | Matiz importante |
|---|---|---|---|
| **Integración continua (CI)** | Los cambios se integran frecuentemente; cada integración ejecuta construcción y pruebas. | Commit, ejecución y resultado de pruebas. | Una prueba verde respalda lo que esa prueba cubre; no demuestra ausencia total de defectos. |
| **Entrega continua (continuous delivery)** | El software queda apto para liberarse tras las verificaciones; puede existir una aprobación para publicar. | Artefacto o versión aprobable y proceso repetible. | El paso a producción puede ser una decisión humana. |
| **Despliegue continuo (continuous deployment)** | Todo cambio que supera las verificaciones llega automáticamente a producción según la política establecida. | Cambio aprobado por puertas automáticas y revisión desplegada. | Exige confianza en pruebas, observación y recuperación. |

La sigla **CD** se usa para los dos últimos conceptos. Conviene escribir el término completo en cada diseño y señalar el punto exacto donde se decide publicar. [Atlassian, F1](#ref-f1); [GitLab, F3](#ref-f3).

~~~mermaid
flowchart LR
    A["Cambio pequeño"] --> B["Commit"]
    B --> C["CI: compilar y probar"]
    C --> D{"¿Pasa criterios?"}
    D -- No --> E["Corregir"]
    E --> B
    D -- Sí --> F["Versión publicable"]
    F --> G{"Política de liberación"}
    G -- Aprobación --> H["Entrega continua"]
    G -- Automática --> I["Despliegue continuo"]
~~~

**Ejemplo.** Un equipo modifica la validación de un pedido. El pull request ejecuta pruebas. Si pasa y se integra a la rama principal, puede: (a) esperar la revisión del responsable para producción, o (b) desplegarse automáticamente. Ambos usan CI; el segundo aplica despliegue continuo.

### 2.2. Git: qué se versiona y qué significa cada paso

Git mantiene instantáneas de archivos y referencias a commits. Hay tres espacios que el estudiante debe distinguir: directorio de trabajo, índice o área de preparación y repositorio local. El repositorio remoto comparte commits y facilita colaboración; GitHub y GitLab alojan repositorios, mientras que Git es el sistema de control de versiones. [Pro Git, F4](#ref-f4).

~~~mermaid
flowchart LR
    A["Archivos de trabajo"] -->|"git add"| B["Índice"]
    B -->|"git commit"| C["Repositorio local"]
    C -->|"git push"| D["Repositorio remoto"]
    D -->|"git fetch"| C
~~~

| Comando | Propósito | Comprobación |
|---|---|---|
| <code>git status</code> | Ver archivos modificados, preparados y rama actual. | Antes de cada commit. |
| <code>git diff</code> y <code>git diff --staged</code> | Revisar cambios sin preparar y preparados. | Detectar datos o archivos agregados por error. |
| <code>git add app.py tests/</code> | Preparar archivos concretos. | Revisar nuevamente el índice. |
| <code>git commit -m "Valida pedidos vacíos"</code> | Guardar una instantánea identificable. | <code>git log --oneline -5</code>. |
| <code>git switch -c feature/validacion</code> | Crear y activar una rama de trabajo. | <code>git branch --show-current</code>. |
| <code>git push -u origin feature/validacion</code> | Compartir la rama. | Comprobar la rama remota. |

**Flujo recomendado de aula:** rama corta → cambio y prueba local → commit → push → pull request → CI → revisión → integración a <code>main</code> → despliegue según política. La revisión puede exigir pruebas y aprobación. Proteja <code>main</code> para que una modificación no eluda el proceso.

**Errores conceptuales frecuentes.** <code>git add</code> no publica; <code>git commit</code> no actualiza GitHub; <code>git push</code> no garantiza un despliegue: solo lo provoca si existe un flujo configurado para ese evento. Un archivo ignorado por <code>.gitignore</code> puede seguir rastreado si ya se incorporó; retirar un secreto de la versión actual tampoco lo borra de la historia. [Pro Git, F4](#ref-f4).

### 2.3. Actividad de la unidad 1

**Reto (parejas, 45-60 min).** Dos personas modifican la misma aplicación: una agrega <code>/health</code> y otra cambia el mensaje de <code>/api/version</code>. Cada una crea su rama, commit y pull request. Antes de integrar, muestran el <code>git diff</code>, la prueba local y el identificador del commit. Luego explican qué sucedería si ambos alteraran la misma línea.

**Preguntas para discusión:** ¿Qué evidencia prueba que el código revisado corresponde al commit desplegado? ¿Puede un resultado verde del pipeline demostrar que la aplicación funciona para todos los usuarios? ¿Qué diferencias hay entre revertir un commit y redirigir tráfico a una revisión anterior?

<a id="unidad-2"></a>
## 3. Unidad 2. Automatización de pipelines

### 3.1. Anatomía de un pipeline

Un **evento** inicia una ejecución; un **workflow** define jobs; cada job corre en un **runner** y contiene pasos. Puede haber dependencias entre jobs, permisos, condiciones, variables y secretos. El estado final depende de las puertas de calidad, no del simple hecho de que se ejecutaron comandos. En GitHub Actions, la configuración vive normalmente en <code>.github/workflows/*.yml</code>; GitLab usa <code>.gitlab-ci.yml</code>; Jenkins conserva el pipeline como código en un <code>Jenkinsfile</code>. [GitHub, F2](#ref-f2); [GitLab, F3](#ref-f3); [Jenkins, F5](#ref-f5).

~~~mermaid
flowchart TD
    PR["Pull request"] --> T["Job: pruebas"]
    PUSH["Push a main"] --> T
    T --> B["Job: construir contenedor"]
    B --> D{"¿Push autorizado?"}
    D -- Sí --> G["Job: desplegar"]
    D -- No --> R["Registrar resultado"]
    G --> V["Verificar revisión"]
    V --> R
~~~

**Diferencia entre paso y puerta.** Ejecutar una prueba que siempre devuelve código 0 no protege la entrega. Una puerta debe tener criterio, salida de error visible y consecuencia: bloquear el job dependiente. Las pruebas unitarias, la construcción del contenedor y una verificación de servicio son capas distintas.

| Herramienta | Archivo y modelo | Ventaja didáctica | Precaución |
|---|---|---|---|
| **GitHub Actions** | YAML; workflows, jobs y steps en GitHub. | Integración directa con repositorio y pull requests. | Definir permisos mínimos de <code>GITHUB_TOKEN</code> y separar CI de credenciales cloud. |
| **GitLab CI/CD** | YAML; jobs, stages y runners. | Variables protegidas y entornos integrados. | Las variables declaradas en YAML son configuración no sensible; los secretos se gestionan fuera del archivo. |
| **Jenkins** | Jenkinsfile; pipeline declarativo o scripted y agentes. | Control de agentes, plugins y entorno propio. | Mantener Jenkins, agentes, plugins y credenciales requiere operación y parches. |

La selección es una decisión de arquitectura y contexto institucional; los tres pueden implementar las mismas ideas de construir, probar y desplegar. [GitLab YAML, F6](#ref-f6); [Jenkins Pipeline, F5](#ref-f5).

### 3.2. Ejemplo mínimo de equivalencia entre plataformas

Los siguientes fragmentos **solo ejecutan pruebas**; no constituyen un despliegue. Sirven para comparar sintaxis:

**GitHub Actions:**

~~~yaml
name: pruebas
on:
  pull_request:
jobs:
  test:
    runs-on: ubuntu-24.04
    permissions:
      contents: read
    steps:
      - uses: actions/checkout@v6
      - uses: actions/setup-python@v6
        with:
          python-version: '3.12'
      - run: python -m unittest discover -s tests -v
~~~

**GitLab CI/CD:**

~~~yaml
stages:
  - test

test:
  stage: test
  image: python:3.12-slim
  script:
    - python -m unittest discover -s tests -v
~~~

**Jenkins, pipeline declarativo:**

~~~groovy
pipeline {
    agent any
    stages {
        stage('Test') {
            steps {
                sh 'python -m unittest discover -s tests -v'
            }
        }
    }
}
~~~

El agente Jenkins del ejemplo debe tener Python instalado; en sistemas distintos de Unix se adapta el paso <code>sh</code>. Los ejemplos de GitHub y GitLab obtienen Python del runner o de una imagen, respectivamente. Las versiones de acciones se verificaron en la documentación de sus repositorios en la fecha indicada; en entornos con requisitos fuertes de integridad se fijan por SHA completo y se actualizan mediante revisión controlada. [GitHub seguridad, F7](#ref-f7).

### 3.3. Pruebas, variables y errores

**Pirámide práctica para el caso del laboratorio:**

1. **Prueba de lógica:** comprueba respuestas para rutas válidas e inválidas. Rápida, sin red.
2. **Prueba de empaquetado:** <code>docker build</code> verifica que el contenedor pueda construirse; por sí sola no demuestra que arranque.
3. **Prueba de humo local:** iniciar el contenedor y solicitar <code>/health</code>.
4. **Verificación de despliegue:** observar revisión lista, ejecutar una solicitud autenticada y revisar métricas/logs.

| Dato | Ejemplo | Ubicación correcta |
|---|---|---|
| Configuración no sensible | ID de proyecto, región, nombre de servicio | Variables de repositorio o entorno. |
| Secreto | Clave de API, contraseña | Gestor de secretos, nunca en el repositorio ni en logs. |
| Identidad de despliegue | Cuenta de servicio cloud | Federación OIDC con permisos mínimos; evitar una llave JSON persistente. |
| Identidad en ejecución | Cuenta de servicio del contenedor | Separada de la identidad que despliega. |

Los secretos ocultos en la interfaz pueden filtrarse si un workflow ejecuta código no confiable o imprime transformaciones que el enmascaramiento no reconoce. No ejecute un job privilegiado con código de un pull request no revisado. Use <code>permissions</code> por job; <code>id-token: write</code> permite solicitar un token OIDC, pero no concede por sí solo acceso a recursos cloud. [GitHub seguridad, F7](#ref-f7); [GitHub OIDC, F8](#ref-f8).

**Manejo de errores:** localizar job y paso, leer la primera causa verificable en el log, reproducir localmente, corregir y volver a ejecutar sobre un commit nuevo. Evite ocultar fallos con <code>|| true</code> o con reintentos indiscriminados. Use reintentos acotados únicamente cuando la falla sea transitoria y el paso sea seguro de repetir. [GitHub logs, F9](#ref-f9).

### 3.4. Actividad de la unidad 2

Construya un pipeline que corra en pull requests y en push a <code>main</code>. Introduzca deliberadamente una prueba fallida. Capture el job rojo, el mensaje de fallo, el commit que corrige y la ejecución posterior en verde. Explique por qué el job de despliegue debe depender del job de pruebas y por qué el pull request no recibe credenciales cloud.

<a id="unidad-3"></a>
## 4. Unidad 3. Despliegue de aplicaciones en la nube

### 4.1. Arquitectura: responsabilidades y límites

**Infraestructura en la nube** ofrece recursos que se pueden aprovisionar bajo demanda. En IaaS se administran más capas (por ejemplo, sistema operativo de una máquina virtual); en una plataforma administrada de contenedores se delegan más tareas de infraestructura. La aplicación sigue siendo responsabilidad del equipo: código, configuración, permisos, datos, pruebas y monitoreo. [Google Cloud, F10](#ref-f10); [AWS, F11](#ref-f11).

El laboratorio usa **Google Cloud Run** como una implementación concreta de la unidad. Los conceptos son transferibles a AWS o Azure, aunque el recurso, IAM y comandos son distintos. Una compilación desde fuente en Cloud Run usa Cloud Build y Artifact Registry para producir y almacenar la imagen; con un Dockerfile, Cloud Build lo utiliza. El servicio Cloud Run crea revisiones y enruta solicitudes a las revisiones que reciben tráfico. [Cloud Run desde fuente, F13](#ref-f13); [Cloud Run revisiones, F14](#ref-f14).

~~~mermaid
flowchart TD
    A["Desarrollador: Git"] --> B["GitHub: PR y main"]
    B --> C["Actions: pruebas"]
    C --> D["OIDC: identidad de despliegue"]
    D --> E["Cloud Build: imagen"]
    E --> F["Artifact Registry"]
    F --> G["Cloud Run: revisión"]
    G --> H["Solicitudes autorizadas"]
    G --> I["Cloud Logging y Monitoring"]
~~~

| Componente | Función | Pregunta de verificación |
|---|---|---|
| Repositorio | Guarda código y configuración del pipeline. | ¿Qué commit se desplegó? |
| Runner de CI | Ejecuta pruebas y construcción de comprobación. | ¿Pasaron pruebas y arranque del contenedor? |
| Federación OIDC | Intercambia identidad del job por credenciales temporales. | ¿Qué repositorio y entorno están autorizados? |
| Cloud Build | Construye desde la fuente durante el despliegue de ejemplo. | ¿Qué fuente y Dockerfile usó? |
| Artifact Registry | Almacena la imagen construida. | ¿A qué revisión corresponde la imagen? |
| Cloud Run | Ejecuta el contenedor y conserva revisiones. | ¿Cuál revisión está lista y recibe tráfico? |
| Logging/Monitoring | Permiten investigar logs y métricas. | ¿Cambió el error o la latencia tras publicar? |

**Distinción clave:** despliegue es crear/configurar una revisión; liberación es asignarle tráfico de usuarios. Una revisión puede existir sin recibir tráfico. Un rollback puede mover tráfico a una revisión anterior sin deshacer el commit de Git. [Cloud Run rollbacks, F15](#ref-f15).

### 4.2. Seguridad de extremo a extremo

1. **Origen:** proteger rama y pull requests; revisar especialmente cambios en workflows.
2. **Runner:** <code>GITHUB_TOKEN</code> con <code>contents: read</code> en jobs que solo leen; dar <code>id-token: write</code> únicamente al job que necesita OIDC.
3. **Federación:** limitar el proveedor a un repositorio y al entorno <code>production</code>; proteger ese entorno en GitHub. OIDC evita almacenar una clave de servicio de larga duración en el repositorio. [Google WIF, F16](#ref-f16); [GitHub OIDC, F8](#ref-f8).
4. **IAM cloud:** cuenta para desplegar y cuenta de ejecución separadas; conceder solamente los roles necesarios.
5. **Servicio:** un despliegue nuevo es privado por defecto; la exposición pública es una decisión independiente. Para el ejercicio se puede probar con un proxy autenticado. [Cloud Run acceso, F17](#ref-f17).
6. **Artefactos y dependencias:** usar imágenes base confiables, revisar actualizaciones y, cuando se exige reproducibilidad fuerte, fijar versiones y digests; el identificador de commit por sí solo no prueba que dos compilaciones binarias sean idénticas. [Docker, F18](#ref-f18); [GitHub seguridad, F7](#ref-f7).
7. **Observación:** no escribir secretos en logs; registrar suficientes datos para correlacionar error, versión y revisión.

### 4.3. Monitoreo, métricas y decisión de recuperación

Cloud Run proporciona métricas de solicitudes, latencia, instancias, CPU y memoria, entre otras, y permite consultar logs por servicio o revisión. Tras desplegar, compare una ventana anterior y posterior: cantidad de solicitudes, errores, latencia y logs del nuevo código. Un endpoint <code>/health</code> que devuelve 200 solo verifica una condición limitada; si la base de datos es indispensable, habrá que diseñar pruebas funcionales específicas sin convertir la salud en una consulta costosa. [Cloud Run Monitoring, F19](#ref-f19); [Cloud Run Logging, F20](#ref-f20).

~~~mermaid
flowchart TD
    A["Nueva revisión"] --> B["Verificar respuesta y métricas"]
    B --> C{"¿Hay degradación?"}
    C -- No --> D["Mantener tráfico y observar"]
    C -- Sí --> E["Detener avance del tráfico"]
    E --> F["Mover tráfico a revisión estable"]
    F --> G["Registrar incidente y corregir"]
~~~

**Medición de desempeño de entrega.** DORA define actualmente cinco métricas: tiempo desde commit hasta producción, frecuencia de despliegue, tiempo de recuperación tras despliegue fallido, proporción de cambios que fallan y proporción de despliegues dedicados a retrabajo no planeado. Una ejecución de pipeline exitosa no equivale a un despliegue exitoso visto por usuarios. [DORA, F12](#ref-f12).

| Métrica | Forma de observarla en un proyecto | Límite de interpretación |
|---|---|---|
| Tiempo de cambio | Hora del commit y hora en que ese cambio sirve producción. | No confundir con duración del job. |
| Frecuencia de despliegue | Cantidad de despliegues a producción por periodo. | Contar revisiones sin tráfico distorsiona la medida. |
| Tasa de fallo de cambios | Despliegues que requirieron intervención inmediata / despliegues totales. | Definir previamente qué constituye una intervención. |
| Recuperación de despliegue fallido | Tiempo desde la falla causada por un despliegue hasta recuperar el servicio. | No cubre necesariamente incidentes ajenos al despliegue. |
| Retrabajo de despliegue | Despliegues no planeados por un incidente / despliegues totales. | Requiere etiquetar la razón del despliegue. |

**Ejemplo de cálculo, datos simulados:** si en un mes se realizan 20 despliegues y 2 requieren rollback o hotfix inmediato, la tasa de fallo de cambios sería 2/20 = 10 %. Esto ilustra la fórmula; no establece una meta universal.

<a id="laboratorio"></a>
## 5. Laboratorio integrador reproducible

**Caso:** una API mínima de versiones y salud para una aplicación de inventario. Se usa solo la biblioteca estándar de Python para concentrar la práctica en el proceso de entrega. El laboratorio tiene seis fases: código, pruebas, contenedor, CI, identidad cloud y despliegue. El despliegue real requiere recursos de Google Cloud y puede generar cargos.

### Fase 1. Crear la aplicación

Estructura en VS Code:

~~~text
demo-ci-cd/
├── app.py
├── tests/
│   └── test_app.py
├── Dockerfile
├── .dockerignore
├── .gcloudignore
├── .gitignore
└── .github/
    └── workflows/
        └── ci-cd.yml
~~~

**Archivo <code>app.py</code>:**

~~~python
import json
import os
from http.server import BaseHTTPRequestHandler, ThreadingHTTPServer
from urllib.parse import urlsplit


def response_for(path):
    if path == "/health":
        return 200, {"status": "ok"}
    if path == "/api/version":
        return 200, {"version": os.getenv("APP_VERSION", "local")}
    return 404, {"error": "not_found"}


class Handler(BaseHTTPRequestHandler):
    def do_GET(self):
        status, payload = response_for(urlsplit(self.path).path)
        body = json.dumps(payload).encode("utf-8")
        self.send_response(status)
        self.send_header("Content-Type", "application/json; charset=utf-8")
        self.send_header("Content-Length", str(len(body)))
        self.end_headers()
        self.wfile.write(body)


if __name__ == "__main__":
    port = int(os.getenv("PORT", "8080"))
    server = ThreadingHTTPServer(("0.0.0.0", port), Handler)
    server.serve_forever()
~~~

Cloud Run inyecta la variable <code>PORT</code>; el proceso debe escuchar en todas las interfaces del contenedor. La respuesta de <code>/api/version</code> identifica la versión lógica que se entrega mediante una variable de entorno; en cloud se usará el SHA del commit como valor. [Contrato de Cloud Run, F21](#ref-f21).

### Fase 2. Probar la lógica

**Archivo <code>tests/test_app.py</code>:**

~~~python
import os
import unittest
from unittest.mock import patch

from app import response_for


class RouteTests(unittest.TestCase):
    def test_health(self):
        self.assertEqual(response_for("/health"), (200, {"status": "ok"}))

    def test_version(self):
        with patch.dict(os.environ, {"APP_VERSION": "abc123"}):
            self.assertEqual(
                response_for("/api/version"),
                (200, {"version": "abc123"}),
            )

    def test_unknown_route(self):
        self.assertEqual(
            response_for("/missing"),
            (404, {"error": "not_found"}),
        )


if __name__ == "__main__":
    unittest.main()
~~~

Desde la raíz del proyecto:

~~~bash
python -m unittest discover -s tests -v
python app.py
~~~

En otra terminal de VS Code:

~~~bash
curl -fsS http://localhost:8080/health
curl -fsS http://localhost:8080/api/version
~~~

**Resultado esperado:** tres pruebas aprobadas; <code>/health</code> retorna <code>{"status":"ok"}</code> y <code>/api/version</code> retorna <code>{"version":"local"}</code>. Detenga el servidor con Ctrl+C. No guarde una captura de pantalla como sustituto del código y del resultado de las pruebas.

### Fase 3. Empaquetar y comprobar el contenedor

**Archivo <code>Dockerfile</code>:**

~~~dockerfile
FROM python:3.12-slim
WORKDIR /app
ENV PYTHONDONTWRITEBYTECODE=1 PYTHONUNBUFFERED=1
COPY app.py .
USER 10001:10001
EXPOSE 8080
CMD ["python", "app.py"]
~~~

**Archivo <code>.dockerignore</code>:**

~~~text
.git
.github
tests
__pycache__
*.pyc
gha-creds-*.json
~~~

**Archivo <code>.gitignore</code>:**

~~~text
__pycache__/
*.pyc
.env
gha-creds-*.json
~~~

**Archivo <code>.gcloudignore</code>:**

~~~text
.git
.gitignore
.github
tests/
__pycache__/
*.pyc
gha-creds-*.json
~~~

La credencial temporal que crea la acción de autenticación queda en el espacio de trabajo del job. La exclusión explícita en <code>.gcloudignore</code> evita incorporarla al paquete de fuente que se envía a Cloud Build; <code>.dockerignore</code> y <code>.gitignore</code> protegen los otros dos contextos. [Google auth, F22](#ref-f22); [Google gcloudignore, F25](#ref-f25).

<code>EXPOSE</code> documenta el puerto del contenedor; la aplicación escucha efectivamente en <code>PORT</code>. La imagen de ejemplo fija la versión mayor y menor de Python, pero la etiqueta <code>python:3.12-slim</code> puede recibir actualizaciones. Para un proceso que exige reproducibilidad binaria, se fija un digest y se programa su actualización. [Docker, F18](#ref-f18).

~~~bash
docker build -t demo-ci-cd:local .
docker run -d --rm --name demo-ci-cd-local -p 8080:8080 demo-ci-cd:local
curl -fsS http://localhost:8080/health
docker stop demo-ci-cd-local
~~~

Si el puerto 8080 está ocupado en el computador, publique otro puerto local, por ejemplo <code>-p 9090:8080</code>, y consulte <code>http://localhost:9090/health</code>.

### Fase 4. Pipeline de CI y despliegue condicionado

Cree <code>.github/workflows/ci-cd.yml</code>:

~~~yaml
name: CI y despliegue
on:
  pull_request:
  push:
    branches: [main]

permissions:
  contents: read

jobs:
  test:
    runs-on: ubuntu-24.04
    steps:
      - uses: actions/checkout@v6
        with:
          persist-credentials: false
      - uses: actions/setup-python@v6
        with:
          python-version: '3.12'
      - name: Pruebas unitarias
        run: python -m unittest discover -s tests -v
      - name: Construir imagen
        run: docker build -t demo-ci-cd:ci .
      - name: Prueba de humo del contenedor
        run: |
          docker run -d --rm --name demo-ci-cd-test -p 8080:8080 demo-ci-cd:ci
          trap 'docker stop demo-ci-cd-test' EXIT
          for attempt in 1 2 3 4 5 6 7 8 9 10; do
            if curl -fsS http://localhost:8080/health; then
              exit 0
            fi
            sleep 1
          done
          exit 1

  deploy:
    if: github.event_name == 'push' && github.ref == 'refs/heads/main'
    needs: test
    runs-on: ubuntu-24.04
    environment: production
    permissions:
      contents: read
      id-token: write
    steps:
      - uses: actions/checkout@v6
        with:
          persist-credentials: false
      - id: auth
        uses: google-github-actions/auth@v3
        with:
          project_id: ${ vars.GCP_PROJECT_ID }}
          workload_identity_provider: ${ vars.WIF_PROVIDER }}
          service_account: ${ vars.DEPLOY_SERVICE_ACCOUNT }}
      - uses: google-github-actions/setup-gcloud@v3
      - name: Desplegar desde fuente
        env:
          GCP_PROJECT_ID: ${ vars.GCP_PROJECT_ID }}
          GCP_REGION: ${ vars.GCP_REGION }}
          RUNTIME_SERVICE_ACCOUNT: ${ vars.RUNTIME_SERVICE_ACCOUNT }}
          APP_VERSION: ${ github.sha }}
        run: |
          gcloud run deploy demo-ci-cd \
            --source . \
            --project="$GCP_PROJECT_ID" \
            --region="$GCP_REGION" \
            --service-account="$RUNTIME_SERVICE_ACCOUNT" \
            --set-env-vars="APP_VERSION=$APP_VERSION" \
            --quiet
          REVISION="$(gcloud run services describe demo-ci-cd \
            --project="$GCP_PROJECT_ID" \
            --region="$GCP_REGION" \
            --format='value(status.latestReadyRevisionName)')"
          test -n "$REVISION"
          echo "Revisión lista: $REVISION"
~~~

**Lectura del workflow.** El job <code>test</code> corre en ambos eventos; <code>deploy</code> exige que <code>test</code> termine bien y que el evento sea un push a <code>main</code>. El entorno <code>production</code> habilita variables y protecciones configuradas en GitHub. El permiso <code>id-token: write</code> está limitado al job de despliegue. Los pasos de autenticación, instalación de CLI y <code>gcloud run deploy --source .</code> siguen las interfaces documentadas por sus proveedores. [GitHub sintaxis, F2](#ref-f2); [Google auth, F22](#ref-f22); [Google setup-gcloud, F23](#ref-f23); [Cloud Run desde fuente, F13](#ref-f13).

**Alcance de la verificación automática:** se prueban código y contenedor localmente; Cloud Build crea otra imagen desde el mismo commit para Cloud Run. Por ello el flujo comprueba el origen, pero no demuestra identidad binaria entre ambas imágenes. Para elevar la garantía, publique una imagen una sola vez en un registro y despliegue esa misma imagen por digest, añadiendo escaneo y constancia de procedencia según las políticas del proyecto.

### Fase 5. Preparar identidad y permisos una sola vez

El administrador ejecuta la preparación inicial en la terminal de Google Cloud, sustituyendo estos **valores de ejemplo** por los suyos. Requiere permisos para habilitar API, crear cuentas de servicio, configurar Workload Identity Federation y administrar IAM. Restrinja el ejercicio a un proyecto de práctica. [Google WIF, F16](#ref-f16); [Cloud Run desde fuente, F13](#ref-f13).

~~~bash
PROJECT_ID="mi-proyecto-de-practica"
GH_REPO="mi-organizacion/mi-repositorio"
GCP_REGION="us-central1"
PROJECT_NUMBER="$(gcloud projects describe "$PROJECT_ID" --format='value(projectNumber)')"
DEPLOY_SA="cicd-deployer@$PROJECT_ID.iam.gserviceaccount.com"
RUNTIME_SA="demo-runtime@$PROJECT_ID.iam.gserviceaccount.com"

gcloud services enable \
  run.googleapis.com cloudbuild.googleapis.com \
  artifactregistry.googleapis.com iam.googleapis.com \
  iamcredentials.googleapis.com sts.googleapis.com \
  --project="$PROJECT_ID"

gcloud iam service-accounts create cicd-deployer --project="$PROJECT_ID"
gcloud iam service-accounts create demo-runtime --project="$PROJECT_ID"

gcloud projects add-iam-policy-binding "$PROJECT_ID" \
  --member="serviceAccount:$DEPLOY_SA" \
  --role="roles/run.sourceDeveloper"
gcloud projects add-iam-policy-binding "$PROJECT_ID" \
  --member="serviceAccount:$DEPLOY_SA" \
  --role="roles/serviceusage.serviceUsageConsumer"
gcloud iam service-accounts add-iam-policy-binding "$RUNTIME_SA" \
  --project="$PROJECT_ID" \
  --member="serviceAccount:$DEPLOY_SA" \
  --role="roles/iam.serviceAccountUser"

gcloud projects add-iam-policy-binding "$PROJECT_ID" \
  --member="serviceAccount:$PROJECT_NUMBER-compute@developer.gserviceaccount.com" \
  --role="roles/run.builder"

gcloud iam workload-identity-pools create github \
  --project="$PROJECT_ID" --location=global \
  --display-name="GitHub Actions"
POOL_ID="$(gcloud iam workload-identity-pools describe github \
  --project="$PROJECT_ID" --location=global --format='value(name)')"

gcloud iam workload-identity-pools providers create-oidc repo-production \
  --project="$PROJECT_ID" --location=global \
  --workload-identity-pool=github \
  --issuer-uri="https://token.actions.githubusercontent.com" \
  --attribute-mapping="google.subject=assertion.sub,attribute.repository=assertion.repository" \
  --attribute-condition="assertion.sub == 'repo:$GH_REPO:environment:production'"

gcloud iam service-accounts add-iam-policy-binding "$DEPLOY_SA" \
  --project="$PROJECT_ID" \
  --role="roles/iam.workloadIdentityUser" \
  --member="principalSet://iam.googleapis.com/$POOL_ID/attribute.repository/$GH_REPO"

WIF_PROVIDER="$(gcloud iam workload-identity-pools providers describe repo-production \
  --project="$PROJECT_ID" --location=global \
  --workload-identity-pool=github --format='value(name)')"
echo "WIF_PROVIDER=$WIF_PROVIDER"
echo "DEPLOY_SERVICE_ACCOUNT=$DEPLOY_SA"
echo "RUNTIME_SERVICE_ACCOUNT=$RUNTIME_SA"
~~~

El proveedor acepta solo la identidad OIDC cuyo <code>sub</code> coincide con ese repositorio y el entorno <code>production</code>; configure ese entorno para que solo <code>main</code> pueda desplegar y, si su plan lo permite, exija revisión. El vínculo de cuenta de servicio se limita al repositorio. La protección de rama y el control de quién modifica workflows son esenciales, porque el <code>sub</code> de entorno no codifica la rama. Los cambios IAM pueden tardar algunos minutos en propagarse. [Google auth, F22](#ref-f22); [GitHub entornos, F24](#ref-f24).

En **GitHub → Settings → Environments → production**, configure la rama autorizada y estas cinco variables de entorno (no son claves secretas):

| Variable | Valor |
|---|---|
| <code>GCP_PROJECT_ID</code> | El ID del proyecto, por ejemplo <code>mi-proyecto-de-practica</code>. |
| <code>GCP_REGION</code> | La región elegida, por ejemplo <code>us-central1</code>. |
| <code>WIF_PROVIDER</code> | Salida completa de <code>WIF_PROVIDER</code> (empieza por <code>projects/</code>). |
| <code>DEPLOY_SERVICE_ACCOUNT</code> | Salida <code>DEPLOY_SERVICE_ACCOUNT</code>. |
| <code>RUNTIME_SERVICE_ACCOUNT</code> | Salida <code>RUNTIME_SERVICE_ACCOUNT</code>. |

**Nota operativa:** el ejemplo usa la cuenta predeterminada de Compute Engine como identidad de Cloud Build; Google documenta el rol <code>roles/run.builder</code> para ella en este flujo. Una organización puede imponer otra identidad de compilación o políticas adicionales: en ese caso ajuste el proyecto antes de ejecutar el pipeline. La cuenta de ejecución creada para la API no recibe permisos de otros servicios cloud porque esta API no los necesita. [Cloud Run desde fuente, F13](#ref-f13).

### Fase 6. Ejecutar, comprobar y recuperar

1. Haga push de una rama de funcionalidad y abra un pull request. Debe correr <code>test</code> y quedar omitido <code>deploy</code>.
2. Integre a <code>main</code> conforme a las protecciones del repositorio. Deben correr <code>test</code> y luego <code>deploy</code>. Guarde commit SHA, URL de la ejecución y nombre de la revisión.
3. En una terminal autenticada de Google Cloud, pruebe el **servicio privado** mediante proxy; la identidad activa debe tener permiso para invocarlo. El proxy se ejecuta en primer plano; deje esa terminal abierta. [Cloud Run acceso, F17](#ref-f17).

~~~bash
gcloud run services proxy demo-ci-cd \
  --project="$PROJECT_ID" --region="$GCP_REGION" --port=9090
~~~

En otra terminal local:

~~~bash
curl -fsS http://localhost:9090/health
curl -fsS http://localhost:9090/api/version
~~~

La segunda respuesta debe contener el SHA del commit integrado. Para revisar el servicio:

~~~bash
gcloud run services describe demo-ci-cd \
  --project="$PROJECT_ID" --region="$GCP_REGION" \
  --format="yaml(status.url,status.latestReadyRevisionName,status.traffic)"
gcloud run services logs read demo-ci-cd \
  --project="$PROJECT_ID" --region="$GCP_REGION" --limit=20
~~~

Si una revisión con tráfico degrada el servicio, identifique en **Revision history** una revisión anterior que funcionaba y cambie el tráfico; sustituya <code>REVISION_ESTABLE</code> por el nombre real:

~~~bash
gcloud run services update-traffic demo-ci-cd \
  --project="$PROJECT_ID" --region="$GCP_REGION" \
  --to-revisions="REVISION_ESTABLE=100"
~~~

**Verifique después del rollback** el destino del tráfico y vuelva a consultar <code>/health</code> y <code>/api/version</code>. Una nueva versión corregida puede requerir devolver explícitamente el tráfico a la revisión más reciente con <code>gcloud run services update-traffic demo-ci-cd --to-latest</code>. Los cambios de tráfico no son instantáneos y las solicitudes en curso completan su procesamiento. [Cloud Run rollbacks, F15](#ref-f15).

### Evidencia mínima del laboratorio

| Fase | Evidencia concreta |
|---|---|
| 1-2 | Código, tres pruebas con resultado y respuesta de <code>/health</code>. |
| 3 | Construcción y solicitud al contenedor en ejecución. |
| 4 | Pull request con CI aprobada y job de despliegue omitido. |
| 5 | Configuración OIDC/variables sin mostrar tokens, claves ni archivos de credenciales. |
| 6 | Ejecución de <code>main</code>, SHA, revisión, respuesta autenticada, logs y explicación de rollback. |

<a id="diagnostico"></a>
## 6. Diagnóstico de fallas y recuperación

| Síntoma | Comprobación prioritaria | Corrección probable |
|---|---|---|
| El workflow no aparece | Ruta <code>.github/workflows/</code>, YAML y evento configurado. | Corregir nombre/ruta o sintaxis; hacer nuevo push. |
| El PR ejecuta pruebas, pero no despliega | Revisar la condición del job. | Es el comportamiento esperado: el despliegue solo se activa tras push a <code>main</code>. |
| <code>python -m unittest</code> falla | Primera aserción fallida y <code>APP_VERSION</code>. | Corregir lógica o expectativa; repetir localmente. |
| <code>docker build</code> falla | Contexto, Dockerfile y referencia de imagen base. | Comprobar archivos presentes, red y contenido del Dockerfile. |
| El contenedor construye, pero no responde | Logs del contenedor; <code>PORT</code>, enlace a <code>0.0.0.0</code> y puerto publicado. | Corregir el proceso o la publicación de puertos. |
| OIDC rechaza el token | <code>WIF_PROVIDER</code>, nombre de repo, entorno y condición <code>sub</code>. | Hacer coincidir identidad real con la política; comprobar propagación IAM. |
| Cloud Build devuelve permiso denegado | Identidad de compilación y rol <code>run.builder</code>. | Conceder el rol a la identidad que realmente construye. |
| Cloud Run no crea revisión | Logs de Cloud Build, permisos del desplegador, Dockerfile y región. | Corregir la causa del primer error, no repetir sin diagnóstico. |
| Respuesta 403 al abrir URL | Política de invocación del servicio privado. | Usar proxy autenticado y verificar permiso <code>run.invoker</code>. |
| Respuesta 200, pero versión inesperada | Valor <code>APP_VERSION</code> y distribución de tráfico por revisión. | Comparar SHA, revisión lista y revisión que recibe tráfico. |
| Aumentan errores tras desplegar | Métricas y logs de la revisión nueva. | Detener avance de tráfico, volver a revisión estable y abrir incidente. |

**Secuencia de investigación:** (1) evento; (2) job; (3) paso; (4) identidad y permisos; (5) construcción; (6) revisión y tráfico; (7) solicitud real; (8) métricas y logs. Un problema de autorización OIDC no se corrige cambiando el puerto de la aplicación; una respuesta 403 de un servicio privado no demuestra que el contenedor falló.

~~~mermaid
sequenceDiagram
    participant E as Estudiante
    participant G as GitHub
    participant A as Actions
    participant C as Cloud Run
    E->>G: Push a rama y abre PR
    G->>A: Evento pull_request
    A-->>G: Pruebas y contenedor
    E->>G: Integra a main
    G->>A: Evento push
    A->>C: Autenticación OIDC y despliegue
    C-->>A: Revisión lista
    A-->>E: Resultado y nombre de revisión
~~~

<a id="actividades"></a>
## 7. Actividades, preguntas y evaluación

### 7.1. Caso de estudio para clase: “Inventario en cierre de jornada”

Una tienda recibe cambios frecuentes en una API de inventario. El viernes se publica una modificación que devuelve 200 en <code>/health</code>, pero responde un valor incorrecto de versión y aumenta el tiempo de respuesta de la función principal. Dos estudiantes forman el equipo de desarrollo y otra pareja hace revisión de operaciones.

**Dinámica (70-90 min):**

1. **Diseñar (15 min):** dibujar el recorrido de un commit hasta usuarios; señalar tres puntos donde un fallo debe detener la entrega.
2. **Ejecutar (25 min):** modificar <code>/api/version</code>, incorporar una prueba y abrir un pull request. El equipo de operaciones predice qué jobs correrán.
3. **Provocar y diagnosticar (15 min):** cambiar temporalmente la expectativa del test; identificar en el log el fallo exacto y repararlo con otro commit.
4. **Simular incidente (15 min):** suponer que la nueva revisión degrada la aplicación pese a que CI fue verde; decidir qué métrica mirar, cómo aislar la revisión y cómo dirigir tráfico a la versión anterior.
5. **Sustentar (10 min):** cada equipo presenta SHA, evidencia de CI, revisión y una limitación de sus pruebas.

**Preguntas de análisis con criterio de respuesta:**

| Pregunta | Elementos que debe incluir una respuesta sólida |
|---|---|
| ¿Por qué <code>/health</code> en verde no invalida el reporte de los usuarios? | El endpoint prueba una condición acotada; hace falta observar función de negocio, latencia y errores. |
| ¿Qué componente crea la imagen en el flujo cloud propuesto? | Cloud Build al desplegar desde fuente; Artifact Registry la almacena. |
| ¿Qué datos permiten relacionar la falla con una entrega? | SHA del commit, ejecución del workflow, revisión, momento del despliegue y métricas/logs por revisión. |
| ¿Cómo se recupera el servicio sin borrar commits? | Reasignar tráfico a una revisión estable y verificar respuesta; después corregir código. |
| ¿Cómo impedir que el código de un PR use credenciales de producción? | Job privilegiado condicionado a push a <code>main</code>, dependencia de pruebas, entorno protegido y política OIDC específica. |
| ¿Cuál es el riesgo de compilar dos veces desde la misma fuente? | Tags y dependencias mutables pueden producir binarios distintos; una política más estricta despliega por digest una imagen construida una vez. |

### 7.2. Retos incrementales

| Nivel | Cambio | Evidencia de aceptación |
|---|---|---|
| 1. Fundamentos | Dos ramas y pull requests sobre la API. | Historial con commits pequeños, diff revisado y CI visible. |
| 2. Pruebas | Agregar un caso 404 y un caso con query string. | Tests que fallan antes de corregir y pasan después; prueba HTTP real. |
| 3. Seguridad | Restringir la identidad OIDC al entorno y proteger <code>main</code>. | Política y ejecución que prueban que un PR no alcanza el job cloud; no mostrar tokens. |
| 4. Operación | Desplegar dos revisiones y provocar un error recuperable. | Revisión estable identificada, tráfico redirigido y verificación posterior. |
| 5. Profundización | Diseñar “construir una vez, desplegar por digest”. | Diagrama de procedencia, registro, digest y permisos; comparación con el laboratorio. |

### 7.3. Autoevaluación breve

1. ¿Cuál es la diferencia entre <code>git commit</code> y <code>git push</code>?
2. ¿En qué punto la entrega continua se convierte en despliegue continuo?
3. ¿Por qué una imagen que construye correctamente puede fallar al arrancar?
4. ¿Qué hace <code>needs: test</code> en el job de despliegue?
5. ¿Por qué el job de pruebas no solicita <code>id-token: write</code>?
6. ¿Cuál es la diferencia entre la cuenta que despliega y la cuenta de ejecución?
7. ¿Un 403 del servicio privado indica necesariamente que falló la aplicación?
8. ¿Qué diferencia existe entre rollback de tráfico y reversión de un commit?
9. ¿Por qué el SHA del commit no demuestra por sí solo identidad binaria?
10. ¿Qué medición de DORA comienza en el commit y termina en producción?

**Clave de discusión:** 1) commit guarda localmente, push envía al remoto; 2) cuando la política publica automáticamente todo cambio que supera las verificaciones; 3) errores de arranque, puerto, configuración o dependencias; 4) exige que el job de pruebas termine bien; 5) no usa identidad cloud; 6) una modifica el despliegue y la otra identifica al proceso en ejecución; 7) no, puede ser falta de autorización; 8) tráfico cambia la revisión servida, Git cambia historia/contenido mediante otro commit; 9) las compilaciones pueden resolver bases o dependencias distintas; 10) tiempo de cambio o *change lead time*.

### 7.4. Relación con la evaluación del syllabus

El documento institucional establece tres cortes: 30 %, 30 % y 40 %. La siguiente tabla hace explícita la contribución al total del curso; no agrega ponderaciones nuevas.

| Corte | Actividad dentro del corte | Peso en el corte | Contribución a la nota final |
|---|---|---:|---:|
| 1 (30 %) | Talleres, exámenes rápidos y exposiciones | 60 % | 18 % |
| 1 (30 %) | Parcial | 40 % | 12 % |
| 2 (30 %) | Talleres, exámenes rápidos y exposiciones | 30 % | 9 % |
| 2 (30 %) | Parcial | 40 % | 12 % |
| 2 (30 %) | Avance de proyecto | 30 % | 9 % |
| 3 (40 %) | Talleres y exámenes rápidos | 20 % | 8 % |
| 3 (40 %) | Avance de proyecto | 20 % | 8 % |
| 3 (40 %) | Presentación de proyecto | 60 % | 24 % |
| **Total** | | | **100 %** |

**Evidencias sugeridas, sin alterar las ponderaciones:** corte 1, análisis de Git/CI y parcial; corte 2, pipeline con pruebas y primer avance; corte 3, despliegue, seguridad, observación, recuperación y sustentación. El syllabus nombra “Parcial”, “Avance de Proyecto” y “Presentación de Proyecto” como entregables; la distribución detallada de los talleres corresponde a la planeación docente.

<a id="fuentes"></a>
## 8. Glosario y fuentes

### 8.1. Glosario esencial

| Término | Definición operativa |
|---|---|
| **Artefacto** | Resultado identificable de una construcción, por ejemplo una imagen de contenedor. |
| **Commit** | Instantánea versionada de cambios en Git, identificada por un hash. |
| **Pull request** | Propuesta de integración que permite revisión y verificaciones automáticas. |
| **Pipeline** | Secuencia o grafo de jobs automatizados con condiciones de ejecución. |
| **Runner** | Entorno donde se ejecuta un job de CI/CD. |
| **Gate / puerta de calidad** | Criterio que permite o bloquea el paso siguiente según evidencia. |
| **Secreto** | Dato sensible que requiere acceso restringido y manejo fuera del código. |
| **OIDC** | Protocolo de identidad que permite a un job demostrar su identidad ante un proveedor cloud. |
| **IAM** | Políticas y roles que controlan quién puede acceder a qué recursos. |
| **Revisión** | Versión inmutable del servicio Cloud Run tras un despliegue o cambio de configuración. |
| **Tráfico** | Porcentaje de solicitudes asignado a cada revisión. |
| **Rollback** | Recuperación hacia una versión operativa anterior; aquí, cambio de tráfico. |
| **Prueba de humo** | Verificación breve de que una funcionalidad crítica responde después de iniciar o publicar. |
| **Observabilidad** | Uso de señales como métricas y logs para entender el estado y diagnosticar fallas. |
| **Digest** | Identificador criptográfico del contenido de una imagen; permite referirse a una imagen concreta. |

### 8.2. Referencias verificables

**Fuente curricular:** Uniempresarial, *Formato Syllabus: Despliegues Continuos*, PDF proporcionado; datos generales y contenidos, pp. 1-3; evaluación, p. 4; bibliografía, p. 5. La fecha de actualización declarada en los datos generales es 17/12/2025; las páginas siguientes muestran pies de formato con otra versión/fecha. Para la planeación académica vigente conviene confirmar el documento controlado con la institución.

Las fuentes técnicas siguientes son documentación de sus proveedores y guías primarias, consultadas el 28/09/2026. Versiones, permisos y precios pueden cambiar; confirme el estado de la documentación antes de una implementación institucional.

- <a id="ref-f1"></a>**F1.** Atlassian, [CI, entrega continua y despliegue continuo](https://www.atlassian.com/continuous-delivery/principles/continuous-integration-vs-delivery-vs-deployment).
- <a id="ref-f2"></a>**F2.** GitHub, [Sintaxis de workflows de GitHub Actions](https://docs.github.com/es/actions/reference/workflows-and-actions/workflow-syntax).
- <a id="ref-f3"></a>**F3.** GitLab, [Fundamentos de CI/CD](https://docs.gitlab.com/ci/).
- <a id="ref-f4"></a>**F4.** Git, [Pro Git: fundamentos y repositorios remotos](https://git-scm.com/book/es/v2/Fundamentos-de-Git-Trabajar-con-Remotos).
- <a id="ref-f5"></a>**F5.** Jenkins, [Pipeline como código](https://www.jenkins.io/doc/book/pipeline/pipeline-as-code/).
- <a id="ref-f6"></a>**F6.** GitLab, [Referencia de sintaxis YAML CI/CD](https://docs.gitlab.com/ci/yaml/).
- <a id="ref-f7"></a>**F7.** GitHub, [Uso seguro de GitHub Actions](https://docs.github.com/en/actions/reference/security/secure-use).
- <a id="ref-f8"></a>**F8.** GitHub, [OpenID Connect para proveedores cloud](https://docs.github.com/en/actions/how-tos/secure-your-work/security-harden-deployments/oidc-in-cloud-providers).
- <a id="ref-f9"></a>**F9.** GitHub, [Logs de ejecuciones](https://docs.github.com/en/actions/how-tos/monitor-workflows/use-workflow-run-logs).
- <a id="ref-f10"></a>**F10.** Google Cloud, [Descripción general de Google Cloud](https://cloud.google.com/docs/overview).
- <a id="ref-f11"></a>**F11.** AWS, [Conceptos de computación en la nube](https://aws.amazon.com/what-is-cloud-computing/).
- <a id="ref-f12"></a>**F12.** DORA, [Métricas de desempeño de entrega](https://dora.dev/guides/dora-metrics/).
- <a id="ref-f13"></a>**F13.** Google Cloud, [Desplegar servicios desde código fuente en Cloud Run](https://docs.cloud.google.com/run/docs/deploying-source-code).
- <a id="ref-f14"></a>**F14.** Google Cloud, [Administrar revisiones de Cloud Run](https://docs.cloud.google.com/run/docs/managing/revisions).
- <a id="ref-f15"></a>**F15.** Google Cloud, [Rollbacks y migración de tráfico](https://docs.cloud.google.com/run/docs/rollouts-rollbacks-traffic-migration).
- <a id="ref-f16"></a>**F16.** Google Cloud, [Workload Identity Federation para pipelines](https://docs.cloud.google.com/iam/docs/workload-identity-federation-with-deployment-pipelines).
- <a id="ref-f17"></a>**F17.** Google Cloud, [Probar servicios privados con proxy](https://docs.cloud.google.com/run/docs/triggering/https-request).
- <a id="ref-f18"></a>**F18.** Docker, [Buenas prácticas de construcción](https://docs.docker.com/build/building/best-practices/).
- <a id="ref-f19"></a>**F19.** Google Cloud, [Métricas y monitoreo de Cloud Run](https://docs.cloud.google.com/run/docs/monitoring).
- <a id="ref-f20"></a>**F20.** Google Cloud, [Logs de Cloud Run](https://docs.cloud.google.com/run/docs/logging).
- <a id="ref-f21"></a>**F21.** Google Cloud, [Contrato del contenedor de Cloud Run](https://docs.cloud.google.com/run/docs/container-contract).
- <a id="ref-f22"></a>**F22.** Google GitHub Actions, [Acción de autenticación y ejemplo de federación](https://github.com/google-github-actions/auth).
- <a id="ref-f23"></a>**F23.** Google GitHub Actions, [Instalar y configurar gcloud en Actions](https://github.com/google-github-actions/setup-gcloud).
- <a id="ref-f24"></a>**F24.** GitHub, [Entornos y reglas de protección](https://docs.github.com/en/actions/reference/workflows-and-actions/deployments-and-environments).
- <a id="ref-f25"></a>**F25.** Google Cloud, [Referencia de <code>.gcloudignore</code>](https://docs.cloud.google.com/sdk/gcloud/reference/topic/gcloudignore).
