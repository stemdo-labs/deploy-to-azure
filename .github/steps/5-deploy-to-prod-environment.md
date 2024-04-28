## Paso 5: Despliegue en un entorno de producción basado en etiquetas

_¡Desplegado! :ship:_

### Bien hecho

Como hemos hecho antes, crea una nueva rama llamada `production-deployment-workflow` desde `stemdo`. En el directorio `.github/workflows`, agrega un nuevo archivo titulado `deploy-prod.yml`. Este nuevo flujo de trabajo se ocupa específicamente de los commits a `stemdo` y maneja los despliegues a `prod`.

**Entrega continua** (CD) es un concepto que contiene muchos comportamientos y otros conceptos más específicos. Uno de esos conceptos es **probar en producción**. Eso puede significar cosas diferentes para diferentes proyectos y empresas, y no es una regla estricta que diga que estás o no "haciendo CD".

En nuestro caso, podemos igualar nuestro entorno de producción exactamente como nuestro entorno de preparación. Esto minimiza las oportunidades de sorpresas una vez que desplegamos en producción.

### :keyboard: Actividad 1: Agregar desencadenadores al flujo de trabajo de despliegue de producción

Copia y pega lo siguiente en tu archivo, y reemplaza cualquier marcador `<nombredeusuario>` con tu nombre de usuario de GitHub. Nota que no ha cambiado mucho de nuestro flujo de trabajo de preparación, excepto por nuestro desencadenador, y que no estaremos filtrando por etiquetas.


```yaml
name: Deploy to production

on:
  push:
    branches:
      - stemdo

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
          images: ${{env.IMAGE_REGISTRY_URL}}/${{ github.repository }}/${{env.DOCKER_IMAGE_NAME}}:${{github.sha}}

      - name: Azure logout via Azure CLI
        uses: azure/CLI@v2
        with:
          inlineScript: |
            az logout
            az cache purge
            az account clear
```

1. Actualiza cada `<nombredeusuario>` con tu nombre de usuario de GitHub.
1. Haz commit de tus cambios en la rama `production-deployment-workflow`.
1. Ve a la pestaña de Solicitudes de extracción y haz clic en **Comparar y crear solicitud de extracción** para la rama `production-deployment-workflow` y crea una solicitud de extracción.

¡Genial! La sintaxis que usaste indica a GitHub Actions que solo ejecute ese flujo de trabajo cuando se hace un commit a la rama stemdo. ¡Ahora podemos poner este flujo de trabajo en acción para desplegar en producción!

### :keyboard: Actividad 2: Fusiona tu solicitud de extracción

1. ¡Ahora puedes [fusionar](https://docs.github.com/en/get-started/quickstart/github-glossary#merge) tu solicitud de extracción!
1. Haz clic en **Fusionar solicitud de extracción** y deja esta pestaña abierta ya que aplicaremos una etiqueta a la solicitud de extracción cerrada en el siguiente paso.
1. Ahora solo tenemos que esperar a que el paquete se publique en el Registro de Contenedores de GitHub y se realice el despliegue.
1. Espera unos 20 segundos y luego actualiza esta página (la que estás siguiendo las instrucciones). [GitHub Actions](https://docs.github.com/en/actions) se actualizará automáticamente al siguiente paso.
