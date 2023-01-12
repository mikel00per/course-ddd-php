📧 Crear módulo de notificaciones
==================================

Como veíamos en el curso de [DDD Aplicado](https://pro.codely.tv/library/domain-driven-design-ddd/87157/about/), puede darse la situación en que veamos necesario que un equipo de desarrollo se centre en una funcionalidad, para lo cual crearíamos un nuevo módulo. Además, si este módulo creciera demasiado podríamos acabar considerando promocionar dicho módulo a Bounded Context.

Veremos un ejemplo de cómo llevar a cabo este proceso con el caso del envío de notificaciones dentro de la plataforma de cursos. Partiremos de la asunción de que necesitamos añadir notificaciones 📫 a nuestro sistema, añadiendo poco a poco más complejidad hasta el punto de necesitar invertir más recursos en él y pasarlo a un Bounded Context propio 🔝

Si pensamos en el caso a nivel de usuario, lo que se plantea es que se podrá elegir diferentes formas de notificación (sms, email, push al escritorio…) ante diferentes eventos de la plataforma (nuevos cursos, respuestas en comentarios…) que será lo que finalmente se traduzca en los diferentes Casos de Uso dentro del módulo de notificaciones

A nivel estructural lo que haremos será crear una nueva carpeta ‘Notifications’ dentro del contexto de Mooc, dentro de la cual y siguiendo con nuestro planteamiento de Arquitectura Hexagonal crearemos las carpetas ‘Domain’, ‘Application’ e ‘Infrastructure’

    src
      |-Backoffice
      |-Mooc
      |   |-Courses
      |   |-CoursesCounter
      |   |-Notifications
      |       |-Application
      |       |-Domain
      |       |-Infrastructure


Esto supone que la infraestructura en la que se apoye el módulo será la del contexto de Mooc, es decir, este equipo será quien asuma las nuevas features de notificaciones

fichero `services.yaml`:

    imports:
      - { resource: ../../../../src/Mooc/Shared/Infrastructure/Symfony/DependencyInjection/mooc_database.yaml }
      - { resource: ../../../../src/Mooc/Shared/Infrastructure/Symfony/DependencyInjection/mooc_services.yaml }
    
    services:
      _defaults:
        autoconfigure: true
        autowire: true
    
      # Configure
      _instanceof:
        CodelyTv\Shared\Domain\Bus\Event\DomainEventSubscriber:
          tags: ['codely.domain_event_subscriber']
    
      CodelyTv\Apps\Mooc\Backend\Controller\:
        resource: '../src/Controller'
        tags: ['controller.service_arguments']
    
      CodelyTv\Apps\Mooc\Backend\Command\:
        resource: '../src/Command'
        tags: ['console.command']
    
      # Wire
      CodelyTv\:
        resource: '../../../../src'
      
      # ...


Gracias a que ya teníamos configurado el Autowire en la aplicación (definiendo como servicios todo lo que se encuentre dentro de /src), de momento no tendremos que realizar ningún tipo de configuración adicional para añadir un nuevo módulo (Una vez que intervengan más contextos veremos como será conveniente modificar esta configuración)

Un aspecto importante en el planteamiento de los distintos casos de uso del módulo es que cada notificación puede requerir un contenido distinto, por lo que tiene todo el sentido darles una lógica distinta, que se traduce en el planteamiento de extraerlo a un Bounded Context

¿Alguna Duda?
=============

Si tienes alguna duda sobre este video o quieres compartir alguna sugerencia no dudes en abrir una nueva discusión más abajo 👇👇👇

¡Nos vemos en el siguiente video: 🆙 Promocionar el módulo notificaciones a Bounded Context!