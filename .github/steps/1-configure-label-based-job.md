## Paso 1: Desencadenar un trabajo basado en etiquetas

_¡Bienvenido al curso! :tada:_

![Captura de pantalla 2022-06-07 a las 4 01 43 PM](https://user-images.githubusercontent.com/6351798/172490466-00f27580-8906-471f-ae83-ef3b6370df7d.png)

Muchas cosas intervienen en la entrega "continua". Estas cosas pueden ir desde la cultura y el comportamiento hasta la automatización específica. En este ejercicio, nos vamos a centrar en la parte de despliegue de nuestra automatización.

En un flujo de trabajo de GitHub Actions, el paso `on` define qué provoca que se ejecute el flujo de trabajo. En este caso, queremos que el flujo de trabajo ejecute diferentes tareas cuando se apliquen etiquetas específicas a una solicitud de extracción.

Usaremos etiquetas como desencadenadores para múltiples tareas:

- Cuando alguien aplique una etiqueta de "iniciar entorno" a una solicitud de extracción, eso le indicará a GitHub Actions que nos gustaría configurar nuestros recursos en un entorno de Azure.
- Cuando alguien aplique una etiqueta de "etapa" a una solicitud de extracción, ese será nuestro indicador de que nos gustaría desplegar nuestra aplicación en un entorno de preparación.
- Cuando alguien aplique una etiqueta de "destruir entorno" a una solicitud de extracción, desmontaremos cualquier recurso que se esté ejecutando en nuestra cuenta de Azure.


### :keyboard: Actividad 1: Configurar los permisos de `GITHUB_TOKEN`

Al comienzo de cada ejecución de flujo de trabajo, GitHub crea automáticamente un secreto único `GITHUB_TOKEN` para usar en tu flujo de trabajo. Necesitamos asegurarnos de que este token tenga los permisos necesarios para este curso.

1. Abre una nueva pestaña del navegador y trabaja en los pasos en tu segunda pestaña mientras lees las instrucciones en esta pestaña.
1. Ve a Configuración > Acciones > General. Asegúrate de que el `GITHUB_TOKEN` también tenga habilitados los **Read and write permissions** bajo **Workflow permissions**. Esto es necesario para que tu flujo de trabajo pueda cargar tu imagen en el registro de contenedores.


### :keyboard: Actividad 2: Configurar un desencadenador basado en etiquetas

Por ahora, nos centraremos en la preparación. Iniciaremos y destruiremos nuestro entorno en un paso posterior.

1. Ve a la pestaña **Actions**.
1. Haz clic en **New workflow**.
1. Busca "flujo de trabajo simple" y haz clic en **Configure**.
1. Nombre tu flujo de trabajo `deploy-staging.yml`.
1. Edita el contenido de este archivo y elimina todos los desencadenadores y trabajos.
1. Edita el contenido del archivo para agregar una condición que filtre el trabajo `build` cuando haya una etiqueta presente llamada **stage**. Tu archivo resultante debería lucir así:

   ```yaml
   name: Stage the app

   on:
     pull_request:
       types: [labeled]

   jobs:
     build:
       runs-on: ubuntu-latest

       if: contains(github.event.pull_request.labels.*.name, 'stage')
   ```

1. Haz clic en **Comenzar commit** y elige crear una nueva rama llamada `flujo-preparación`.
1. Haz clic en **Proponer cambios**.
1. Haz clic en **Crear solicitud de extracción**.
1. Espera unos 20 segundos y luego actualiza esta página (la que estás siguiendo las instrucciones). [GitHub Actions](https://docs.github.com/en/actions) se actualizará automáticamente al siguiente paso.
