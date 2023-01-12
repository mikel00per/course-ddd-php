Relación entre módulos User y Auth
==================================

Siguiendo con el tema de autenticación de usuarios, otra pregunta bastante interesante es si resulta necesario inyectar el `UserRepository` (BC de `User`) cada vez que autenticamos a un usuario (BC `Auth`)

En la pregunta anterior veíamos cómo podíamos introducir un **Middleware** para el tema de la autenticación delante de los controladores que nos devolviera el `UserId`, lo que podría plantearse como una alternativa, si bien puede no ser muy explícita con el uso de decoradores que oculten lo que está sucediendo

Una opción por la que nos decantaríamos para evitar tener que llamar al contexto de User cada vez que queremos comprobar el nombre del usuario desde el contexto de Auth es **mantener una proyección** de esa información en la **BD de Auth** vía eventos de dominio (Ej. UserRegistered)

Por otro lado, también podríamos optar por introducir un **CommandBus y QueryBus en el Controller** de forma que lanzaríamos en primer lugar un comando síncrono para la autenticación y en segundo lugar una query para recuperar el token generado y devolverlo en la respuesta al cliente

¿Alguna Duda?
=============

Si tienes alguna duda sobre el contenido del video o quieres compartir alguna sugerencia no dudes en abrir una nueva discusión más abajo 👇👇👇

¡Nos vemos en la última lección: Conclusiones y Siguientes pasos!