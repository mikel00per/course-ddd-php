🙆‍♂️ Error al publicar eventos: Fallback en MySql
==================================================

Un tema importante ahora que gestionamos las acciones ‘derivadas’ a través de la publicación de eventos es que estos siempre se publiquen para que puedan ser posteriormente consumidos por sus correspondientes casos de uso subscritos. Pero al llevar a cabo este proceso mediante un sistema de colas como RabbitMQ nos encontramos con que no tenemos una garantía absoluta de que la publicación se lleve a cabo, por lo que se hace necesario contar con un sistema de Fallback para que, en caso de fallo, no perdamos dichos eventos

Lo que haremos será publicar en MySQL aquellos eventos que por alguna razón no hayan podido hacerlo mediante RabbitMQ, aprovechando la tabla de BD y la implementación del EventBus de MySQL que ya teníamos del paso previo

Clase `RabbitMQEventBus`:

    final class RabbitMqEventBus implements EventBus
    {
        private $connection;
        private $exchangeName;
        private $failoverPublisher;
        public function __construct(
            RabbitMqConnection $connection,
            string $exchangeName,
            MySqlDoctrineEventBus $failoverPublisher
        ) {
            $this->connection        = $connection;
            $this->exchangeName      = $exchangeName;
            $this->failoverPublisher = $failoverPublisher;
        }
        public function publish(DomainEvent ...$events): void
        {
            each($this->publisher(), $events);
        }
        private function publisher(): callable
        {
            return function (DomainEvent $event) {
                try {
                    $this->publishEvent($event);
                } catch (AMQPException $error) {
                    $this->failoverPublisher->publish($event);
                }
            };
        }
        private function publishEvent(DomainEvent $event): void
        {
            $body       = DomainEventJsonSerializer::serialize($event);
            $routingKey = $event::eventName();
            $messageId  = $event->eventId();
            $this->connection->exchange($this->exchangeName)->publish(
                $body,
                $routingKey,
                AMQP_NOPARAM,
                [
                    'message_id'       => $messageId,
                    'content_type'     => 'application/json',
                    'content_encoding' => 'utf-8',
                ]
            );
        }
    }


El refactor del eventBus será bastante sencillo: englobaremos la publicación en el exchange dentro de un bloque try-catch de modo que si falla lo publique desde el EventBus de MySQL. Aprovechamos que los tres posibles errores que puede lanzar la librería de AMQP heredan de la misma Exception padre para definir ésta en el Catch 👨‍👦‍👦, asegurándonos que sea esa condición bajo la cual se publique en MySQL

Puesto que tenemos claro que queremos irnos a MySQL, no nos importa que lo que reciba el constructor sea la implementación ⛓ de `MySqlDoctrineEventBus` y no una interface

¿Alguna Duda?
=============

Si tienes alguna duda sobre este video o quieres compartir alguna sugerencia no dudes en abrir una nueva discusión más abajo 👇👇👇

¡Nos vemos en la siguiente lección: 🐰 Consumir eventos con RabbitMQ!