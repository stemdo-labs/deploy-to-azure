## Paso 2: Configurar un entorno de Azure

_¡Buen trabajo por comenzar! :gear:_

### Buen trabajo al desencadenar un trabajo en etiquetas específicas

No entraremos en detalle en los pasos de este flujo de trabajo, pero sería una buena idea familiarizarse con las acciones que estamos utilizando. Son:


- [`actions/checkout`](https://github.com/actions/checkout)
- [`actions/upload-artifact`](https://github.com/actions/upload-artifact)
- [`actions/download-artifact`](https://github.com/actions/download-artifact)
- [`docker/login-action`](https://github.com/docker/login-action)
- [`docker/build-push-action`](https://github.com/docker/build-push-action)
- [`azure/login`](https://github.com/Azure/login)
- [`azure/webapps-deploy`](https://github.com/Azure/webapps-deploy)



### :keyboard: Actividad 1: Almacena tus credenciales en secretos de GitHub y termina de configurar tu flujo de trabajo

1. En una pestaña nueva, [crea una cuenta en Azure](https://azure.microsoft.com/en-us/free/) si aún no tienes una. Si tu cuenta de Azure se creó a través del trabajo, es posible que encuentres problemas para acceder a los recursos necesarios; recomendamos crear una nueva cuenta para uso personal y para este curso.
    > **Nota**: Es posible que necesites una tarjeta de crédito para crear una cuenta en Azure. Si eres estudiante, es posible que también puedas aprovechar el [Paquete de Desarrollador para Estudiantes](https://education.github.com/pack) para acceder a Azure. Si deseas continuar con el curso sin una cuenta en Azure, Skills seguirá respondiendo, pero ninguno de los despliegues funcionará.
1. Crea una [nueva suscripción](https://docs.microsoft.com/en-us/azure/cost-management-billing/manage/create-subscription) en el Portal de Azure.
    > **Nota**: tu suscripción debe configurarse como "Pagar según se usa", lo que requerirá que ingreses información de facturación. Este curso solo utilizará unos pocos minutos de tu plan gratuito, pero Azure requiere la información de facturación.
1. Instala [Azure CLI](https://docs.microsoft.com/en-us/cli/azure/install-azure-cli?view=azure-cli-latest) en tu máquina.

1.  En tu terminal, ejecuta:
    ```shell
    az login
    ```

1. Copia el valor del campo `id:` a un lugar seguro. Lo llamaremos `AZURE_SUBSCRIPTION_ID`. Aquí tienes un ejemplo de cómo se ve:

    ```shell
    [
    {
      "cloudName": "AzureCloud",
      "id": "f****a09-****-4d1c-98**-f**********c", # <-- Copy this id field
      "isDefault": true,
      "name": "some-subscription-name",
      "state": "Enabled",
      "tenantId": "********-a**c-44**-**25-62*******61",
      "user": {
        "name": "mdavis******@*********.com",
        "type": "user"
        }
      }
    ]
    ```
1. En tu terminal, ejecuta el siguiente comando.
    ````shell
    az ad sp create-for-rbac --name "GitHub-Actions" --role contributor \
     --scopes /subscriptions/{subscription-id} \
     --sdk-auth

        # Replace {subscription-id} with the same id stored in AZURE_SUBSCRIPTION_ID.
        ```

    > **Note**: The `\` character works as a line break on Unix based systems. If you are on a Windows based system the `\` character will cause this command to fail. Place this command on a single line if you are using Windows.\*\*

    ````

1. Copia todo el contenido de la respuesta del comando, lo llamaremos `AZURE_CREDENTIALS`. Aquí tienes un ejemplo de cómo se ve:

    ```shell
    {
      "clientId": "<GUID>",
      "clientSecret": "<GUID>",
      "subscriptionId": "<GUID>",
      "tenantId": "<GUID>",
      (...)
    }
    ```

1. De regreso en GitHub, haz clic en **Secrets and variables > Actions** en la pestaña de Configuración.
1. Haz clic en **New repository secret**.
1. Nombra tu nuevo secreto **AZURE_SUBSCRIPTION_ID** y pega el valor del campo `id:` del primer comando.
1. Haz clic en **Add secret**.
1. Haz clic en **New repository secret** nuevamente.
1. Nombra el segundo secreto **AZURE_CREDENTIALS** y pega todo el contenido del segundo comando que ingresaste en la terminal.
1. Haz clic en **Add secret**.
1. Regresa a la pestaña de Solicitudes de extracción y en tu solicitud de extracción ve a la pestaña **Files Changed**. Encuentra y luego edita el archivo `.github/workflows/deploy-staging.yml` para usar algunas acciones nuevas.

El archivo completo del flujo de trabajo debería lucir así:


```yaml
name: Deploy to staging

on:
  pull_request:
    types: [labeled]

env:
  IMAGE_REGISTRY_URL: ghcr.io
  ###############################################
  ### Replace <username> with GitHub username ###
  ###############################################
  DOCKER_IMAGE_NAME: <username>-azure-ttt
  AZURE_WEBAPP_NAME: <username>-ttt-app
  ###############################################

jobs:
  build:
    if: contains(github.event.pull_request.labels.*.name, 'stage')

    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: 16
      - name: npm install and build webpack
        run: |
          npm install
          npm run build
      - uses: actions/upload-artifact@v4
        with:
          name: webpack artifacts
          path: public/

  Build-Docker-Image:
    runs-on: ubuntu-latest
    needs: build
    name: Build image and store in GitHub Container Registry
    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Download built artifact
        uses: actions/download-artifact@v4
        with:
          name: webpack artifacts
          path: public

      - name: Log in to GHCR
        uses: docker/login-action@v3
        with:
          registry: ${{ env.IMAGE_REGISTRY_URL }}
          username: ${{ github.actor }}
          password: ${{ secrets.GITHUB_TOKEN }}

      - name: Extract metadata (tags, labels) for Docker
        id: meta
        uses: docker/metadata-action@v5
        with:
          images: ${{env.IMAGE_REGISTRY_URL}}/${{ github.repository }}/${{env.DOCKER_IMAGE_NAME}}
          tags: |
            type=sha,format=long,prefix=

      - name: Build and push Docker image
        uses: docker/build-push-action@v5
        with:
          context: .
          push: true
          tags: ${{ steps.meta.outputs.tags }}
          labels: ${{ steps.meta.outputs.labels }}

  Deploy-to-Azure:
    runs-on: ubuntu-latest
    needs: Build-Docker-Image
    name: Deploy app container to Azure
    steps:
      - name: "Login via Azure CLI"
        uses: azure/login@v2
        with:
          creds: ${{ secrets.AZURE_CREDENTIALS }}

      - uses: azure/docker-login@v1
        with:
          login-server: ${{env.IMAGE_REGISTRY_URL}}
          username: ${{ github.actor }}
          password: ${{ secrets.GITHUB_TOKEN }}

      - name: Deploy web app container
        uses: azure/webapps-deploy@v3
        with:
          app-name: ${{env.AZURE_WEBAPP_NAME}}
          images: ${{env.IMAGE_REGISTRY_URL}}/${{ github.repository }}/${{env.DOCKER_IMAGE_NAME}}:${{ github.sha }}

      - name: Azure logout via Azure CLI
        uses: azure/CLI@v2
        with:
          inlineScript: |
            az logout
            az cache purge
            az account clear
```

16. Después de haber editado el archivo, haz clic en **Confirmar cambios...** y confirma en la rama `staging-workflow`.
17. Espera unos 20 segundos y luego actualiza esta página (la que estás siguiendo las instrucciones). [GitHub Actions](https://docs.github.com/en/actions) se actualizará automáticamente al siguiente paso.
