## Paso 4: Despliegue en un entorno de preparación basado en etiquetas

_¡Bien hecho, has utilizado un flujo de trabajo para iniciar tu entorno de Azure! :dancer:_

Ahora que la configuración adecuada y los archivos de flujo de trabajo están presentes, ¡vamos a probar nuestras acciones! En este paso, hay un pequeño cambio en el juego. ¡Una vez que agregues la etiqueta apropiada a tu solicitud de extracción, deberías poder ver el despliegue!

1. Crea una nueva rama llamada `staging-test` desde `stemdo` utilizando los mismos pasos que hiciste para la rama anterior `azure-configuration`.
1. Edita el archivo `.github/workflows/deploy-staging.yml`, y reemplaza cada `<nombredeusuario>` con tu nombre de usuario de GitHub.
1. Haz commit de ese cambio en la nueva rama `staging-test`.
1. Ve a la pestaña de Solicitudes de extracción y debería haber un banner amarillo con la rama `staging-test` para `Comparar y crear solicitud de extracción`. Una vez que se abra la solicitud de extracción, haz clic en `Crear solicitud de extracción`.

### :keyboard: Actividad 1: Agrega la etiqueta adecuada a tu solicitud de extracción

1. Asegúrate de que el `GITHUB_TOKEN` para este repositorio tenga permisos de lectura y escritura en **Permisos de flujo de trabajo**. [Aprende más](https://docs.github.com/en/actions/security-guides/automatic-token-authentication#modifying-the-permissions-for-the-github_token). Esto es necesario para que tu flujo de trabajo pueda cargar tu imagen en el registro de contenedores.
1. Crea y aplica la etiqueta `stage` a tu solicitud de extracción abierta.
1. Espera a que se ejecute el flujo de trabajo de GitHub Actions y despliegue la aplicación en tu entorno de Azure. Puedes seguir el progreso en la pestaña de Acciones o en el cuadro de fusión de la solicitud de extracción. El despliegue puede tardar unos momentos, pero has hecho lo correcto. Una vez que el despliegue sea exitoso, verás marcas de verificación verdes para cada ejecución, y verás una URL para tu despliegue. ¡Juega el juego!
1. Espera unos 20 segundos y luego actualiza esta página (la que estás siguiendo las instrucciones). [GitHub Actions](https://docs.github.com/en/actions) se actualizará automáticamente al siguiente paso.
