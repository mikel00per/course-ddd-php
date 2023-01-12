🕳 Dónde comprobar permisos de usuarios y qué info añadir en los eventos
===========================================================================

Una de las formas más básicas de añadir una capa de seguridad en la comunicación con el backend de nuestra aplicación es añadiendo una ‘Basic Auth HTTP’, permitiéndonos controlar qué usuarios pueden acceder a el mediante peticiones HTTP

En términos de DDD es importante establecer dónde se va a llevar a cabo ese proceso de Autorización ya que precisamente lo que queremos es comprobar si el usuario Autenticado tiene los permisos suficientes (está Autorizado) para ejecutar un caso de uso 👮

De este modo, asumiremos que si se publica un evento de ‘Curso creado’ será porque previamente se ha comprobado que el usuario puede publicar cursos y, por tanto, no será necesario añadir dentro del evento de dominio ninguna metadata adicional para que los subscriptores comprueben si el usuario puede o no realizar esta acción

En el caso en que las acciones derivadas estuvieran condicionadas por el tipo de usuario que realizó la acción del evento, si que sería en el propio subscriptor donde haríamos dicha comprobación y/o recuperar cualquier información adicional que pueda requerir (en ningún caso condicionaremos la información del productor del evento por las necesidades de los subscriptores de ese evento)

¿Alguna Duda?
=============

Si tienes alguna duda sobre este video o quieres compartir alguna sugerencia no dudes en abrir una nueva discusión más abajo 👇👇👇

¡Nos vemos en el siguiente video: ✋ Implementación con basic auth HTTP!