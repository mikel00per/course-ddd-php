🚀 Planteamiento pipeline del deploy a producción
==================================================

Hemos desarrollado nuestro proyecto y lo tenemos listo en el repo, pero a la hora de desplegarlo en tres servidores diferentes ¿Cómo lo hacemos? Existen varias estrategias que podemos seguir a la hora de hacer el deploy a producción

Normalmente los primeros pasos de nuestra pipeline consistirán en bajar las dependencias necesarias y preparar (y securizar) la caché para preparar el entorno. Una vez preparado, existen distintas posibilidades

*   Desplegar todas las aplicaciones 🚚
*   Desplegar solo aquellas en las que se hayan producido cambios 🕵

A nivel de directorios es interesante seguir estas premisas:

*   Se ha modificado código en la carpeta _/apps_: Tendremos que desplegar esa aplicación
*   Se ha modificado código en la carpeta _/src_ de algún contexto: Tendremos que desplegar todas las aplicaciones que dependan de ese contexto (o todas si no tenemos un mapping de aplicación-contextos)
*   Se ha modificado código en la carpeta _/shared_: Tendremos que desplegar todas las aplicaciones

¿Alguna Duda?
=============

Si tienes alguna duda sobre este video o quieres compartir alguna sugerencia no dudes en abrir una nueva discusión más abajo 👇👇👇

¡Nos vemos en el siguiente Video: 🚶 Conclusión y siguientes pasos!