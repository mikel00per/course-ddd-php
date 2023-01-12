🗣️ Cache de cliente: ETag, max expire, y demás amigos de HTTP
==============================================================

Tanto ETag como max expire son elementos que viajan en los headers de los mensajes HTTP y que nos permiten controlar desde el lado del cliente cuándo pedir de nuevo contenido al servidor y cuando utilizar el que tengamos almacenado en caché sin tener que realizar una nueva llamada

*   ETag: Envía un hash del contenido de la petición de modo que podamos comprobar si el contenido ha cambiado desde la última petición (y enviar un 304 en caso de ser el mismo)
*   max expire: Establece cuándo debe el cliente volver a realizar la petición para solicitar de nuevo el contenido

El objeto Response de Symfony nos provee ya de setters específicos para establecer estos campos en el header sin que tengamos que preocuparnos por su implementación

El punto de interés de este tema es, **siguiendo DDD dónde estableceríamos** las comprobaciones asociadas a estos elementos. Puesto que se trata de algo completamente ligado al protocolo de comunicación HTTP **corresponde al Controlador** o, en caso de querer mantenerlo más limpio, extraerlo a un middleware que situemos por delante del propio controlador

¿Alguna Duda?
=============

Si tienes alguna duda sobre este video o quieres compartir alguna sugerencia no dudes en abrir una nueva discusión más abajo 👇👇👇

¡Nos vemos en el siguiente video: 🦐 Alternativas de cache en servidor!