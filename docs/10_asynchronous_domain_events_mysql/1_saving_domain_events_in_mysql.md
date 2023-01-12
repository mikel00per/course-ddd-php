🥡 Guardado de eventos de dominio en MySQL
==========================================

Hemos visto cómo llevar a cabo la **publicación de eventos de dominio de forma síncrona** y, aunque de este modo conseguíamos el propósito de separar la acción principal (Crear un curso) de las acciones derivadas (Incrementar el contador de cursos, enviar una notificación…), nos encontramos con el problema de bloquear el flujo hasta que no terminan estas acciones derivada: Estamos bloqueando al usuario 🙅‍♂

Una primera aproximación para la publicación asíncrona de los eventos podría ser almacenar los eventos en una base de datos como MySQL para consumirlos posteriormente. Este paso no sólo mejorará la performance en el flujo de la acción principal, sino que también nos va a simplificar bastante la gestión de errores de esos eventos derivados

*   Podéis encontrar el código relativo a este video en la [rama principal del repo](https://github.com/CodelyTV/php-ddd-skeleton/)

Clase `MySqlDoctrineEventBus`:

    final class MySqlDoctrineEventBus implements EventBus
    {
        private const DATABASE_TIMESTAMP_FORMAT = 'Y-m-d H:i:s';
        private $connection;
        public function __construct(EntityManager $entityManager)
        {
            $this->connection = $entityManager->getConnection();
        }
        public function publish(DomainEvent ...$domainEvents): void
        {
            each($this->publisher(), $domainEvents);
        }
        private function publisher(): callable
        {
            return function (DomainEvent $domainEvent): void {
                $id          = $this->connection->quote($domainEvent->eventId());
                $aggregateId = $this->connection->quote($domainEvent->aggregateId());
                $name        = $this->connection->quote($domainEvent::eventName());
                $body        = $this->connection->quote(Utils::jsonEncode($domainEvent->toPrimitives()));
                $occurredOn  = $this->connection->quote(
                    Utils::stringToDate($domainEvent->occurredOn())->format(self::DATABASE_TIMESTAMP_FORMAT)
                );
                $this->connection->executeUpdate(
                    <<<SQL
                    INSERT INTO domain_events (id,  aggregate_id, name,  body,  occurred_on) 
                                       VALUES ($id, $aggregateId, $name, $body, $occurredOn);
    SQL
                );
            };
        }
    }


Esta implementación del EventBus recibirá por constructor la misma instancia de `EntityManager` que venimos utilizando en este Bounded Context. El método _publish()_ al que llamaremos desde el caso de uso recibirá por su parte una lista con todos los eventos de dominio y, por cada evento estaremos llamando al callable _publisher()_

Este _publisher()_ preparará en primer lugar los atributos con los datos recibidos en el evento y seguidamente ejecutará la sentencia SQL para persistir el evento en BD. En este punto recurriremos al método _toPrimitives()_ de la clase `DomainEvent` para meter en el atributo _body_ el array de atributos particulares de ese evento de dominio. El atributo _occuredOn_ lo formatearemos al estándar de MySQL para poder insertarlo (Aunque esto nos afectará en tiempo de tests como veremos más adelante)

Hay que señalar que todos los atributos han pasado por la función _quote()_ de nuestro EntityManager para escapar los valores y evitar así conflictos al ejecutar la query

Finalmente tendremos que especificar en nuestro fichero de configuración que queremos usar ésta implementación de `EventBus` y no otra. Para nuestros tests, por otra parte, mantendremos la especificación de que debe inyectar la implementación ‘InMemory’ puesto que no querremos atacar a la BD en el test unitario


    imports:
      - { resource: ../../../../src/Mooc/Shared/Infrastructure/Symfony/DependencyInjection/mooc_services.yaml }
    
    services:
      _defaults:
        autoconfigure: true
        autowire: true
    
      # Configure
      _instanceof:
        CodelyTv\Shared\Domain\Bus\Event\DomainEventSubscriber:
          tags: ['codely.domain_event_subscriber']
    
        CodelyTv\Shared\Domain\Bus\Command\CommandHandler:
          tags: ['codely.command_handler']
    
        CodelyTv\Shared\Domain\Bus\Query\QueryHandler:
          tags: ['codely.query_handler']
    
        # ...
    
      # -- IMPLEMENTATIONS SELECTOR --
      CodelyTv\Shared\Domain\Bus\Event\EventBus: '@CodelyTv\Shared\Infrastructure\Bus\Event\MySql\MySqlDoctrineEventBus'


¿Alguna Duda?
=============

Si tienes alguna duda sobre este video o quieres compartir alguna sugerencia no dudes en abrir una nueva discusión más abajo 👇👇👇

¡Nos vemos en el siguiente video: 👾 Consumir eventos de dominio desde MySQL!