馃ァ Guardado de eventos de dominio en MySQL
==========================================

Hemos visto c贸mo llevar a cabo la **publicaci贸n de eventos de dominio de forma s铆ncrona** y, aunque de este modo consegu铆amos el prop贸sito de separar la acci贸n principal (Crear un curso) de las acciones derivadas (Incrementar el contador de cursos, enviar una notificaci贸n鈥?), nos encontramos con el problema de bloquear el flujo hasta que no terminan estas acciones derivada: Estamos bloqueando al usuario 馃檯鈥嶁檪

Una primera aproximaci贸n para la publicaci贸n as铆ncrona de los eventos podr铆a ser almacenar los eventos en una base de datos como MySQL para consumirlos posteriormente. Este paso no s贸lo mejorar谩 la performance en el flujo de la acci贸n principal, sino que tambi茅n nos va a simplificar bastante la gesti贸n de errores de esos eventos derivados

*   Pod茅is encontrar el c贸digo relativo a este video en la [rama principal del repo](https://github.com/CodelyTV/php-ddd-skeleton/)

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


Esta implementaci贸n del EventBus recibir谩 por constructor la misma instancia de `EntityManager` que venimos utilizando en este Bounded Context. El m茅todo _publish()_ al que llamaremos desde el caso de uso recibir谩 por su parte una lista con todos los eventos de dominio y, por cada evento estaremos llamando al callable _publisher()_

Este _publisher()_ preparar谩 en primer lugar los atributos con los datos recibidos en el evento y seguidamente ejecutar谩 la sentencia SQL para persistir el evento en BD. En este punto recurriremos al m茅todo _toPrimitives()_ de la clase `DomainEvent` para meter en el atributo _body_ el array de atributos particulares de ese evento de dominio. El atributo _occuredOn_ lo formatearemos al est谩ndar de MySQL para poder insertarlo (Aunque esto nos afectar谩 en tiempo de tests como veremos m谩s adelante)

Hay que se帽alar que todos los atributos han pasado por la funci贸n _quote()_ de nuestro EntityManager para escapar los valores y evitar as铆 conflictos al ejecutar la query

Finalmente tendremos que especificar en nuestro fichero de configuraci贸n que queremos usar 茅sta implementaci贸n de `EventBus` y no otra. Para nuestros tests, por otra parte, mantendremos la especificaci贸n de que debe inyectar la implementaci贸n 鈥業nMemory鈥? puesto que no querremos atacar a la BD en el test unitario


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


驴Alguna Duda?
=============

Si tienes alguna duda sobre este video o quieres compartir alguna sugerencia no dudes en abrir una nueva discusi贸n m谩s abajo 馃憞馃憞馃憞

隆Nos vemos en el siguiente video: 馃懢 Consumir eventos de dominio desde MySQL!