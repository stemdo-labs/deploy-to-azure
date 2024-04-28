## Paso 6: Despliegue en producción

_¡Buen trabajo! :sparkle:_

¡Excelente trabajo, lo has logrado! Deberías poder ver la imagen de tu contenedor en la sección **Paquetes** de tu cuenta en la página del repositorio de stemdo. Puedes obtener la URL de despliegue en el registro de acciones, al igual que la URL de preparación.

### El entorno en la nube

A lo largo del curso, has creado recursos que, si se dejan sin supervisión, podrían generar facturación o consumir tus minutos gratuitos del proveedor de la nube. Una vez que hayas verificado tu aplicación en producción, ¡desmontemos esos entornos para que puedas conservar tus minutos para seguir aprendiendo!


### :keyboard: Actividad 1: Destruye cualquier recurso en ejecución para que no incurras en cargos

1. Crea y aplica la etiqueta `destroy environment` a tu solicitud de extracción fusionada `production-deployment-workflow`. Si ya has cerrado la pestaña con tu solicitud de extracción, puedes abrirla nuevamente haciendo clic en **Solicitudes de extracción** y luego haciendo clic en el filtro **Cerradas** para ver las solicitudes de extracción fusionadas.

Ahora que has aplicado la etiqueta adecuada, esperemos a que el flujo de trabajo de GitHub Actions se complete. Cuando haya terminado, puedes confirmar que tu entorno ha sido destruido visitando la URL de tu aplicación o iniciando sesión en el portal de Azure para verificar que no esté en ejecución.

2. Espera unos 20 segundos y luego actualiza esta página (la que estás siguiendo las instrucciones). [GitHub Actions](https://docs.github.com/en/actions) se actualizará automáticamente al siguiente paso.
