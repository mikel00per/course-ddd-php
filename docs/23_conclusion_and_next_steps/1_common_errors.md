🤭 Errores comunes
==================

Eventos en lugar de Comandos 🤔
-------------------------------

Un error que a veces se comete es el de publicar Eventos que realmente son Comandos. Por ejemplo `SendUserRegistrationEmailDomainEvent`, donde ya de base encontramos que la acción no está en pasado (no estamos reflejando una acción que ya ha sucedido y que es inmutable). En este caso lo que debería haber es un evento `UserRegistered` y que un subscriber que esté escuchándolo sea el que realice la acción de enviar el email

El problema de este tipo de Eventos es que estamos conociendo las acciones derivadas dentro del caso de uso primario, y cuando queramos añadir cualquier otra acción derivada, tendremos que volver al caso de uso primario y añadir otro nuevo evento, violando el principio OCP de SOLID, por lo que debemos invertir esta responsabilidad de conocer las acciones derivadas y mantenerla en los propios subscriptores

Formato de Eventos no estandarizado 🗣
--------------------------------------

Otro escenario que puede darse cuando empezamos a aplicar DDD en una empresa es que los distintos equipos se hayan definido distintos formatos de eventos de dominio, bien por utilizar diferentes lenguajes (Json, XML…) o por utilizar diferentes nombres de atributos o estructuras. A la hora de querer escuchar eventos del otro equipo tendremos que utilizar conversores y controlar si recibimos unos atributos u otros

Por esta razón es importante establecer desde el principio un formato estándar (tanto a nivel de estructura como de nomenclatura), concensuado por todos los equipos, de modo que cualquiera pueda almacenar, recuperar o consumir estos eventos sin que se produzcan conflictos

Eventos genéricos 🌐
--------------------

También puede ocurrir que se definan Eventos de Dominio muy genéricos y de una granularidad muy grande (UserChanged, VideoUpdated) que se lancen por tanto ante diferentes acciones (renombrar el usuario, modificar el apellido, actualizar la imagen de perfil…) y que sea dentro del propio evento donde se especifique qué campos se han modificado

La alternativa sería definir Eventos de menor granularidad y más específicos (UserRenamed, UserProfilePhotoUpdated…) que se traduciría en una mejor semántica. Sin embargo esta opción nos obligaría a que si, por ejemplo, el usuario modifica varios campos a la vez, tendremos que publicar múltiples eventos de dominio (aumentando el coste en el sistema de colas)

Al final tomar una opción u otra será un trade-off y dependerá de cual nos interesa más dado nuestro contexto

¿Alguna Duda?
=============

Si tienes alguna duda sobre este video o quieres compartir alguna sugerencia no dudes en abrir una nueva discusión más abajo 👇👇👇