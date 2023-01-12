Integración de Symfony Messenger y adapter interface EventBus
==============================================================

Comenzamos la lección de Eventos de Dominio y lo haremos viendo cómo instalar Symfony Messenger dentro de nuestra aplicación. Avanzaremos pasito a pasito, creando inicialmente un EventBus síncrono que pese a no ofrecernos la mejor performance si que nos ayudará a mantener el principio SOLID de SRP 🕺

*   Podéis encontrar el código relativo a este video en el tag [0.15.0 del repo](https://github.com/CodelyTV/php-ddd-skeleton/tree/0.15.0)

Añadimos Symfony Messenger a nuestro composer.json del mismo modo que con el resto de nuestras dependencias:

    composer require symfony/messenger


Clase `SymfonySyncEventBus` (En el video como InMemorySymfonyEventBus):

    class SymfonySyncEventBus implements EventBus
    {
        private $bus;
    
    public function __construct(iterable $subscribers)
        {
            $this->bus = new MessageBus(
                [
                    new HandleMessageMiddleware(
                        new HandlersLocator(
                            CallableFirstParameterExtractor::forPipedCallables($subscribers)
                        )
                    ),
                ]
            );
        }
        public function notify(DomainEvent $event): void
        {
            $this->bus->dispatch($event);
        }
    }


Como cabe esperar, nuestra clase hereda de `EventBus` y lo que hará en el constructor será instanciar un nuevo `MessageBus`, propio de Symfony Messenger. Este MessageBus espera recibir un array de Middleware, aunque de momento sólo le estaremos pasando uno, `HandleMessageMiddleware`, que simplemente ejecutará lo que haya dentro.

El Middleware por su parte recibirá un `HandlersLocatorInterface` por parámetro (siguiendo el [locator pattern](https://martinfowler.com/articles/injection.html#UsingAServiceLocator)) que, a su vez, le estaremos pasando un array asociativo de domainEvent->subscribers que nos proporciona nuestro [forPipedCallables](https://github.com/CodelyTV/php-ddd-skeleton/blob/443d47e8a7b5fdf5bcf9bfe1117255a90bdac791/src/Shared/Infrastructure/Bus/CallableFirstParameterExtractor.php#L33-L36)

Este array vendría a tener una estructura como la siguiente:

    $subscribers = [IncreaseCoursesCounterOnCourseCreated, SendEmailOnUserRegistered, LogAllDomainEventsOnEventsOcurred]
    [
    CourseCreated::class => [IncreaseCoursesCounterOnCourseCreated, LogAllDomainEventsOnEventsOcurred ]
    UserRegisteredCreated::class => [SendEmailOnUserRegistered, LogAllDomainEventsOnEventsOcurred ]
    ]


Como vemos, lo que recibe nuestro EventBus por constructor es un array con las instancias de todo lo que hayamos definido como event subscribers. Lo que hará entonces nuestro `forPipedCallables` será ver a qué eventos está subscrito cada uno de ellos y crear un HashMap como el que vemos.

Para llevar a cabo esta magia es necesario que llevemos a cabo algunas configuraciones en el fichero services.yaml

Fichero `services.yaml`:

    services:
      _defaults:
        autoconfigure: true
        autowire: true
    
      _instanceof:
        CodelyTv\Shared\Domain\Bus\DomainEventSubscriber:
          tags: ['codely.domain_event_subscriber']
    
      # ....
    
      CodelyTv\Shared\Infrastructure\Bus\Event\SymfonySyncEventBus:
        arguments: [!tagged codely.domain_event_subscriber]


Lo que estamos indicándole es, por un lado, que todo aquello que sea instancia de la interface `DomainEventSubscriber` se le añada el tag indicado. Luego, por otro lado, le estamos indicamos que nos instancie en nuestro `SymfonySyncEventBus` todo lo que tenga ese tag. Además, recordemos que el autoconfigure debe estár establecido a true.

El contrato de la interface `DomainEventSubscriber` nos obliga a implementar el método estático _subscribedTo()_ con el que tendremos que indicar la lista de eventos de dominio a los que está subscrito 🗣

De vuelta a nuestro EventBus señalar que hemos añadido un [control en la publicación de los eventos](https://github.com/CodelyTV/php-ddd-skeleton/blob/24a44a3a9578aee2307de1d990a3dc6324e4c429/src/Shared/Infrastructure/Bus/Event/InMemory/InMemorySymfonyEventBus.php#L32), que a pesar de ser un Anti-Pattern, necesitamos tenerlo para evitar que pete en caso de tener un evento de dominio sin subscriptores (debido a la interpretación propia de Symfony Messenger)

¿Alguna Duda?
=============

Si tienes alguna duda sobre este video o quieres compartir alguna sugerencia no dudes en abrir una nueva discusión más abajo 👇👇👇

¡Nos vemos en el siguiente video: 🥳 Publicación y suscripción de eventos de dominio y test de aceptación!