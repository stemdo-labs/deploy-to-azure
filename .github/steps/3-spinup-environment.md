## Paso 3: Iniciar un entorno basado en etiquetas

_¡Bien hecho! :heart:_

GitHub Actions es agnóstico en la nube, por lo que cualquier nube funcionará. Mostraremos cómo desplegar en Azure en este curso.

**¿Qué son _los recursos de Azure_?** En Azure, un recurso es una entidad gestionada por Azure. Utilizaremos los siguientes recursos de Azure en este curso:

- Una [aplicación web](https://docs.microsoft.com/en-us/azure/app-service/overview) es cómo desplegaremos nuestra aplicación en Azure.
- Un [grupo de recursos](https://docs.microsoft.com/en-us/azure/azure-resource-manager/management/overview#resource-groups) es una colección de recursos, como aplicaciones web y máquinas virtuales (VMs).
- Un [plan de App Service](https://docs.microsoft.com/en-us/azure/app-service/overview-hosting-plans) es lo que ejecuta nuestra aplicación web y gestiona la facturación (nuestra aplicación debería ejecutarse de forma gratuita).

A través del poder de GitHub Actions, podemos crear, configurar y destruir estos recursos a través de nuestros archivos de flujo de trabajo.


### :keyboard: Actividad 1: Configurar un token de acceso personal (PAT)

Los tokens de acceso personal (PAT) son una alternativa al uso de contraseñas para la autenticación en GitHub. Utilizaremos un PAT para permitir que tu aplicación web extraiga la imagen del contenedor después de que tu flujo de trabajo envíe una imagen recién creada al registro.

1. Abre una nueva pestaña del navegador y trabaja en los pasos en tu segunda pestaña mientras lees las instrucciones en esta pestaña.
2. Crea un token de acceso personal con los ámbitos `repo` y `read:packages`. Para obtener más información, consulta ["Creación de un token de acceso personal"](https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/creating-a-personal-access-token)
3. Una vez que hayas generado el token, necesitaremos almacenarlo en un secreto para que pueda ser utilizado dentro de un flujo de trabajo. Crea un nuevo secreto del repositorio llamado `CR_PAT` y pega el token PAT como valor.
4. Con esto hecho, podemos pasar a configurar nuestro flujo de trabajo.


**Configuring your Azure environment**

To deploy successfully to our Azure environment:

1. Create a new branch called `azure-configuration` by clicking on the branch dropdown on the top, left hand corner of the `Code` tab on your repository page.
2. Once you're in the new `azure-configuration` branch, go into the `.github/workflows` directory and create a new file titled `spinup-destroy.yml` by clicking **Add file**.

Copy and paste the following into this new file:

**Configurando tu entorno de Azure**

Para desplegar con éxito en nuestro entorno de Azure:

1. Crea una nueva rama llamada `azure-configuration` haciendo clic en el menú desplegable de ramas en la esquina superior izquierda de la pestaña `Code` en la página de tu repositorio.
2. Una vez que estés en la nueva rama `cazure-configuration`, ve al directorio `.github/workflows` y crea un nuevo archivo titulado `spinup-destroy.yml` haciendo clic en **Añadir archivo**.

Copia y pega lo siguiente en este nuevo archivo:


```yaml
name: Configure Azure environment

on:
  pull_request:
    types: [labeled]

env:
  IMAGE_REGISTRY_URL: ghcr.io
  AZURE_RESOURCE_GROUP: cd-with-actions
  AZURE_APP_PLAN: actions-ttt-deployment
  AZURE_LOCATION: '"East US"'
  ###############################################
  ### Replace <username> with GitHub username ###
  ###############################################
  AZURE_WEBAPP_NAME: <username>-ttt-app

jobs:
  setup-up-azure-resources:
    runs-on: ubuntu-latest
    if: contains(github.event.pull_request.labels.*.name, 'spin up environment')
    steps:
      - name: Checkout repository
        uses: actions/checkout@v4

      - name: Azure login
        uses: azure/login@v2
        with:
          creds: ${{ secrets.AZURE_CREDENTIALS }}

      - name: Create Azure resource group
        if: success()
        run: |
          az group create --location ${{env.AZURE_LOCATION}} --name ${{env.AZURE_RESOURCE_GROUP}} --subscription ${{secrets.AZURE_SUBSCRIPTION_ID}}

      - name: Create Azure app service plan
        if: success()
        run: |
          az appservice plan create --resource-group ${{env.AZURE_RESOURCE_GROUP}} --name ${{env.AZURE_APP_PLAN}} --is-linux --sku F1 --subscription ${{secrets.AZURE_SUBSCRIPTION_ID}}

      - name: Create webapp resource
        if: success()
        run: |
          az webapp create --resource-group ${{ env.AZURE_RESOURCE_GROUP }} --plan ${{ env.AZURE_APP_PLAN }} --name ${{ env.AZURE_WEBAPP_NAME }}  --deployment-container-image-name nginx --subscription ${{secrets.AZURE_SUBSCRIPTION_ID}}

      - name: Configure webapp to use GHCR
        if: success()
        run: |
          az webapp config container set --docker-custom-image-name nginx --docker-registry-server-password ${{secrets.CR_PAT}} --docker-registry-server-url https://${{env.IMAGE_REGISTRY_URL}} --docker-registry-server-user ${{github.actor}} --name ${{ env.AZURE_WEBAPP_NAME }} --resource-group ${{ env.AZURE_RESOURCE_GROUP }} --subscription ${{secrets.AZURE_SUBSCRIPTION_ID}}

  destroy-azure-resources:
    runs-on: ubuntu-latest

    if: contains(github.event.pull_request.labels.*.name, 'destroy environment')

    steps:
      - name: Checkout repository
        uses: actions/checkout@v4

      - name: Azure login
        uses: azure/login@v2
        with:
          creds: ${{ secrets.AZURE_CREDENTIALS }}

      - name: Destroy Azure environment
        if: success()
        run: |
          az group delete --name ${{env.AZURE_RESOURCE_GROUP}} --subscription ${{secrets.AZURE_SUBSCRIPTION_ID}} --yes
```

3. Haz clic en **Confirmar cambios...** y selecciona `Commit directly to the azure-configuration branch.` antes de hacer clic en **Confirmar cambios**.
4. Ve a la pestaña de Solicitudes de extracción del repositorio.
5. Debería haber un banner amarillo con la rama `azure-configuration` donde puedes hacer clic en **Comparar y crear solicitud de extracción**.
6. Establece el título de la solicitud de extracción como: `Added spinup-destroy.yml workflow` y haz clic en `Crear solicitud de extracción`.

A continuación, cubriremos la funcionalidad clave y luego pondremos en uso el flujo de trabajo aplicando una etiqueta a la solicitud de extracción.

Este nuevo flujo de trabajo tiene dos trabajos:

1. **Configurar recursos de Azure** se ejecutará si la solicitud de extracción contiene una etiqueta con el nombre "iniciar entorno".
2. **Destruir recursos de Azure** se ejecutará si la solicitud de extracción contiene una etiqueta con el nombre "destruir entorno".

Además de cada trabajo, hay algunas variables de entorno globales:

- `AZURE_RESOURCE_GROUP`, `AZURE_APP_PLAN` y `AZURE_WEBAPP_NAME` son nombres para nuestro grupo de recursos, plan de servicio de la aplicación y aplicación web, respectivamente, a los que haremos referencia en múltiples pasos y flujos de trabajo.
- `AZURE_LOCATION` nos permite especificar la [región](https://azure.microsoft.com/en-us/global-infrastructure/regions/) para los centros de datos, donde finalmente se desplegará nuestra aplicación.


**Configuración de recursos de Azure**

El primer trabajo configura los recursos de Azure de la siguiente manera:

1. Inicia sesión en tu cuenta de Azure con la acción [`azure/login`](https://github.com/Azure/login). El secreto `AZURE_CREDENTIALS` que creaste anteriormente se utiliza para la autenticación.
1. Crea un [grupo de recursos de Azure](https://docs.microsoft.com/en-us/azure/azure-resource-manager/management/overview#resource-groups) ejecutando [`az group create`](https://docs.microsoft.com/en-us/cli/azure/group?view=azure-cli-latest#az-group-create) en la CLI de Azure, que está [preinstalada en el runner hospedado de GitHub](https://help.github.com/en/actions/reference/software-installed-on-github-hosted-runners).
1. Crea un [plan de servicio de la aplicación](https://docs.microsoft.com/en-us/azure/app-service/overview-hosting-plans) ejecutando [`az appservice plan create`](https://docs.microsoft.com/en-us/cli/azure/appservice/plan?view=azure-cli-latest#az-appservice-plan-create) en la CLI de Azure.
1. Crea una [aplicación web](https://docs.microsoft.com/en-us/azure/app-service/overview) ejecutando [`az webapp create`](https://docs.microsoft.com/en-us/cli/azure/webapp?view=azure-cli-latest#az-webapp-create) en la CLI de Azure.
1. Configura la nueva aplicación web para usar [GitHub Packages](https://help.github.com/en/packages/publishing-and-managing-packages/about-github-packages) utilizando [`az webapp config`](https://docs.microsoft.com/en-us/cli/azure/webapp/config?view=azure-cli-latest) en la CLI de Azure. Azure se puede configurar para usar su propio [Registro de contenedores de Azure](https://docs.microsoft.com/en-us/azure/container-registry/), [DockerHub](https://docs.docker.com/docker-hub/) o un registro personalizado (privado). En este caso, configuraremos GitHub Packages como un registro personalizado.


**Destrucción de recursos de Azure**

El segundo trabajo destruye los recursos de Azure para que no utilices tus minutos gratuitos ni incurras en facturación. El trabajo funciona de la siguiente manera:

1. Inicia sesión en tu cuenta de Azure con la acción [`azure/login`](https://github.com/Azure/login). El secreto `AZURE_CREDENTIALS` que creaste anteriormente se utiliza para la autenticación.
2. Elimina el grupo de recursos que creamos anteriormente utilizando [`az group delete`](https://docs.microsoft.com/en-us/cli/azure/group?view=azure-cli-latest#az-group-delete) en la CLI de Azure.

### :keyboard: Actividad 2: Aplicar etiquetas para crear recursos

1. Edita el archivo `spinup-destroy.yml` en tu solicitud de extracción abierta y reemplaza cualquier marcador `<username>` con tu nombre de usuario de GitHub. Haz commit a este cambio directamente en la rama `azure-configuration`.
2. De vuelta en la solicitud de extracción, crea y aplica la etiqueta `spin up environment` a tu solicitud de extracción abierta.
3. Espera a que se ejecute el flujo de trabajo de GitHub Actions y se inicie tu entorno de Azure. Puedes seguir el progreso en la pestaña de Acciones o en el cuadro de fusión de la solicitud de extracción.
4. Espera unos 20 segundos y luego actualiza esta página (la que estás siguiendo las instrucciones). [GitHub Actions](https://docs.github.com/en/actions) se actualizará automáticamente al siguiente paso.

Haz lo mismo.
