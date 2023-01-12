✋ Implementación con basic auth HTTP
=====================================

Una premisa importante a la hora de añadir estos elementos en nuestra aplicación es que debemos modelar nosotros mismos el concepto de ‘Autorización’ como algo que forma parte de nuestro Dominio

En este caso queremos partir de una implementación más tosca sobre la cual iteraremos para optimizar la autorización. Para ello colocaremos un **Middleware por delante del Controller** que, en colaboración con el propio framework, gestione esta lógica 👮‍♀

Definiendo nuestro Middleware 🙋‍♂️
-----------------------------------

Configuración en `services.yaml`:

    CodelyTv\Shared\Infrastructure\Symfony\BasicHttpAuthMiddleware:
      tags:
        - { name: kernel.event_listener, event: kernel.request, method: onKernelRequest }


Estamos definiendo este middleware en el fichero de definición de servicios, indicándole que cuando se reciba una Request (aquí vemos la primera colaboración con el framework) se ejecute el [método](https://github.com/CodelyTV/php-ddd-skeleton/blob/d67e903ddbc3f01d888bd636ea715387667aea32/src/Shared/Infrastructure/Symfony/BasicHttpAuthMiddleware.php#L25) que le indicamos

Una vez en este método, lo primero que comprueba es si esta `RequestEvent` requiere que el usuario esté autenticado, para lo cual revisa si trae el atributo ‘auth’ que nosotros mismos hemos definido el el router dentro del parámetro `default`

En caso de requerir autenticación, lo siguiente que hacemos es recuperar el usuario y contraseña desde las cabeceras de la petición (Cuando PHP identifica que hay una petición basic auth HTTP guarda estos dos campos en la cabecera automágicamente 🧙‍♂️)

En base a estas credenciales comprobamos si el usuario los ha introducido para cortar el flujo de la petición y solicitarlos en caso de que no se hayan enviado o, en caso afirmativo, tratar de autenticarlo

En caso de que las credenciales sean correctas, además de permitir continuar con el flujo de la petición hacia el controller, estamos añadiendo el usuario a la propia petición para que el controller pueda disponer de él si necesita realizar cualquier acción (por ejemplo validar que este usuario autenticado tenga autorización para ejecutar el caso de uso asociado)

En el caso de definir un segundo Middleware para gestionar la Autorización del usuario, tendríamos que establecer que éste sería siempre dependiente de haberse ejecutado satisfactoriamente el de Autenticación

¿Alguna Duda?
=============

Si tienes alguna duda sobre este video o quieres compartir alguna sugerencia no dudes en abrir una nueva discusión más abajo 👇👇👇

¡Nos vemos en el siguiente video: 👌 Modelando el concepto “Auth”!