# Examen de entrenamiento: CI/CD, GitHub Actions y despliegue en la nube

**Modalidad:** completar un término por pregunta. **Valor sugerido:** 1 punto por pregunta. Responde primero los diez enunciados y consulta después la clave.

## Preguntas

### Unidad 1. Fundamentos de CI/CD y control de versiones

1. Un estudiante corrige la API `demo-ci-cd` y registra ese cambio en la historia de Git. El registro tiene un identificador SHA. Ese registro se llama **__________**.

2. La corrección está en la rama `mejora-inventario`. Antes de integrarla a `main`, se abre en GitHub una propuesta para revisar los cambios y sus verificaciones. Esa propuesta se llama **__________**.

3. Cada vez que se abre o actualiza esa propuesta, GitHub Actions ejecuta automáticamente las pruebas del código. Esta práctica corresponde a la **__________**.

### Unidad 2. Automatización de pipelines

4. El archivo YAML `.github/workflows/ci.yml` define eventos y jobs que GitHub Actions ejecuta. La unidad de automatización definida por ese archivo se llama **__________**.

5. En `runs-on: ubuntu-24.04`, GitHub Actions selecciona el entorno que ejecutará los pasos del job. Ese entorno de ejecución se denomina **__________**.

6. El job `deploy` debe esperar a que `test` termine correctamente. Completa la clave YAML: `__________: test`.

7. La región de despliegue `GCP_REGION` es una configuración no sensible del repositorio. Completa su referencia en GitHub Actions: `${{ __________.GCP_REGION }}`.

### Unidad 3. Despliegue en la nube

8. El job de despliegue utiliza el token OIDC de GitHub para obtener acceso temporal a Google Cloud sin guardar una clave JSON permanente. El mecanismo de federación de Google Cloud se llama **__________** (puedes escribir su sigla).

9. En el ejemplo `gcloud run deploy demo-ci-cd --source .`, el servicio que construye la imagen de contenedor desde el código fuente es **__________**.

10. Cloud Run crea una versión del servicio con una imagen y configuración determinadas. Esa versión, que puede recibir una proporción del tráfico, se llama **__________**.

## Respuestas y justificaciones

| N.º | Respuesta esperada | Justificación |
|---:|---|---|
| 1 | **Commit** | Es el registro de cambios en Git; su SHA identifica ese estado de la historia. |
| 2 | **Pull request** | Es la propuesta de integración que permite revisar el cambio y observar los checks antes del merge. |
| 3 | **Integración continua (CI)** | CI verifica automáticamente cambios que se integran con frecuencia. Ejecutar pruebas ante un pull request es una manifestación de ese proceso; no equivale por sí solo a desplegar. |
| 4 | **Workflow** o **flujo de trabajo** | GitHub Actions define un workflow mediante un archivo YAML en `.github/workflows/`, con eventos y uno o más jobs. |
| 5 | **Runner** | El runner es la máquina o entorno donde se ejecuta un job. `ubuntu-24.04` selecciona una imagen de runner alojado por GitHub. |
| 6 | **`needs`** | La dependencia `needs: test` hace que `deploy` espere el resultado de `test`; bajo el comportamiento normal, un fallo del job requerido impide avanzar. |
| 7 | **`vars`** | El contexto `vars` expone variables de configuración definidas en GitHub. La expresión completa es `${{ vars.GCP_REGION }}`; los valores sensibles se gestionan de otra forma, por ejemplo con `secrets`. |
| 8 | **Workload Identity Federation (WIF)** | WIF establece la confianza para intercambiar la identidad OIDC de GitHub por acceso autorizado en Google Cloud. `id-token: write` permite solicitar el token, pero no concede por sí solo permisos cloud. |
| 9 | **Cloud Build** | En el despliegue desde fuente, Cloud Build produce la imagen; Artifact Registry la almacena. La imagen local construida en CI no es necesariamente el mismo binario. |
| 10 | **Revisión** | Una revisión representa una versión del servicio Cloud Run. El tráfico puede asignarse a una revisión nueva o regresar a una anterior durante un rollback. |

## Fuentes oficiales

- [GitHub Docs: acerca de Git](https://docs.github.com/es/get-started/using-git/about-git) y [pull requests](https://docs.github.com/en/pull-requests/collaborating-with-pull-requests/proposing-changes-to-your-work-with-pull-requests/about-pull-requests).
- [GitHub Docs: workflows y sintaxis de GitHub Actions](https://docs.github.com/en/actions/reference/workflows-and-actions/workflow-syntax), [jobs](https://docs.github.com/en/actions/how-tos/write-workflows/choose-what-workflows-do/use-jobs) y [variables](https://docs.github.com/en/actions/reference/workflows-and-actions/variables).
- [GitHub Docs: OIDC en proveedores cloud](https://docs.github.com/en/actions/how-tos/secure-your-work/security-harden-deployments/oidc-in-cloud-providers) y [Google Cloud: federación de identidad para pipelines](https://docs.cloud.google.com/iam/docs/workload-identity-federation-with-deployment-pipelines).
- [Google Cloud: despliegue de Cloud Run desde fuente](https://docs.cloud.google.com/run/docs/deploying-source-code) y [administración de revisiones](https://docs.cloud.google.com/run/docs/managing/revisions).

