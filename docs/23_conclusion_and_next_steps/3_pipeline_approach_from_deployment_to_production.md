馃殌 Planteamiento pipeline del deploy a produccio虂n
==================================================

Hemos desarrollado nuestro proyecto y lo tenemos listo en el repo, pero a la hora de desplegarlo en tres servidores diferentes 驴C贸mo lo hacemos? Existen varias estrategias que podemos seguir a la hora de hacer el deploy a producci贸n

Normalmente los primeros pasos de nuestra pipeline consistir谩n en bajar las dependencias necesarias y preparar (y securizar) la cach茅 para preparar el entorno. Una vez preparado, existen distintas posibilidades

*   Desplegar todas las aplicaciones 馃殮
*   Desplegar solo aquellas en las que se hayan producido cambios 馃暤

A nivel de directorios es interesante seguir estas premisas:

*   Se ha modificado c贸digo en la carpeta _/apps_: Tendremos que desplegar esa aplicaci贸n
*   Se ha modificado c贸digo en la carpeta _/src_ de alg煤n contexto: Tendremos que desplegar todas las aplicaciones que dependan de ese contexto (o todas si no tenemos un mapping de aplicaci贸n-contextos)
*   Se ha modificado c贸digo en la carpeta _/shared_: Tendremos que desplegar todas las aplicaciones

驴Alguna Duda?
=============

Si tienes alguna duda sobre este video o quieres compartir alguna sugerencia no dudes en abrir una nueva discusi贸n m谩s abajo 馃憞馃憞馃憞

隆Nos vemos en el siguiente Video: 馃毝 Conclusio虂n y siguientes pasos!