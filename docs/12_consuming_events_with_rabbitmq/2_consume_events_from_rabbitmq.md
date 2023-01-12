🌬 Consumir eventos desde RabbitMQ
==================================

Al igual que veíamos en lecciones anteriores con MySQL, la consumición de eventos se lleva a cabo a través de un servicio. Al encontrarnos con que las diferencias entre ambos consumidores van más allá de los detalles de infraestructura, sino que atañen al propio enfoque de cómo vamos a proceder, no vemos necesidad ni beneficio al hecho de tratar de extraer una interface común como veíamos en el caso de los repositorios en otros puntos del curso

Clase `RabbitMqDomainEventsConsumer`:

    final class RabbitMqDomainEventsConsumer
    {
        private $connection;
        private $deserializer;
        private $exchangeName;
        private $maxRetries;
    
        public function __construct(
            RabbitMqConnection $connection,
            DomainEventJsonDeserializer $deserializer,
            string $exchangeName,
            int $maxRetries
        ) {
            $this->connection   = $connection;
            $this->deserializer = $deserializer;
            $this->exchangeName = $exchangeName;
            $this->maxRetries   = $maxRetries;
        }
    
        public function consume(callable $subscriber, string $queueName): void
        {
            try {
                $this->connection->queue($queueName)->consume($this->consumer($subscriber));
            } catch (AMQPQueueException $error) {
                // We don't want to raise an error if there are no messages in the queue
            }
        }
    
        private function consumer(callable $subscriber): callable
        {
            return function (AMQPEnvelope $envelope, AMQPQueue $queue) use ($subscriber) {
                $event = $this->deserializer->deserialize($envelope->getBody());
                try {
                    $subscriber($event);
                } catch (Throwable $error) {
                    $this->handleConsumptionError($envelope, $queue);
                    throw $error;
                }
                $queue->ack($envelope->getDeliveryTag());
            };
        }
    
      // ...
    }


Desde nuestro _consume()_ vamos utilizar la conexión que recibíamos en el constructor para recuperar la cola que estamos recibiendo y consumir el callable _consumer()_. Este consumer es un wrapper en el que vamos a deserializar el evento de dominio para después ejecutar el subscriber y una vez realizado hacer ack del evento para borrarlo de la cola (Si se produce un error ejecutando el subscriber no llegará a este punto y permanecerá en la cola)

### Testando el consumidor de eventos ✅

Test `RabbitMqEventBusTest` (Resultado final):

    final class RabbitMqEventBusTest extends InfrastructureTestCase
    {
        
        private $connection;
        private $exchangeName;
        private $configurer;
        private $publisher;
        private $consumer;
        private $fakeSubscriber;
        private $consumerHasBeenExecuted;
        
        protected function setUp(): void
        {
            parent::setUp();
            $this->connection = $this->service(RabbitMqConnection::class);
            $this->exchangeName            = 'test_domain_events';
            $this->configurer              = new RabbitMqConfigurer($this->connection);
            
            $this->publisher               = new RabbitMqEventBus(
                $this->connection,
                $this->exchangeName,
                $this->service(MySqlDoctrineEventBus::class)
            );
            
            $this->consumer                = new RabbitMqDomainEventsConsumer(
                $this->connection,
                $this->service(DomainEventJsonDeserializer::class),
                $this->exchangeName,
                $maxRetries = 1
            );
            
            $this->fakeSubscriber          = new TestAllWorksOnRabbitMqEventsPublished();
            $this->consumerHasBeenExecuted = false;
            $this->cleanEnvironment($this->connection);
        }
        
        /** @test */
        public function it_should_publish_and_consume_domain_events_from_rabbitmq(): void
        {
            $domainEvent = CourseCreatedDomainEventMother::random();
            $this->configurer->configure($this->exchangeName, $this->fakeSubscriber);
            $this->publisher->publish($domainEvent);
            
            $this->consumer->consume(
                $this->assertConsumer($domainEvent),
                RabbitMqQueueNameFormatter::format($this->fakeSubscriber)
            );
        
            $this->assertTrue($this->consumerHasBeenExecuted);
        }
        
        // ...
        
        private function assertConsumer(DomainEvent ...$expectedDomainEvents): callable
        {
            return function (DomainEvent $domainEvent) use ($expectedDomainEvents): void {
                $this->assertContainsEquals($domainEvent, $expectedDomainEvents);
                $this->consumerHasBeenExecuted = true;
            };
        }
    
        // ...
    }


Puesto que el _consume()_ de nuestro consumidor de RabbitMQ lo que espera recibir es un callable (y no necesariamente un DomainEventSubscriber) y el nombre de la cola, lo que haremos en primer lugar pasarle un _assertConsumer()_ (En el video spyConsummer, se ha modificado para poder jugar tanto con caso de acierto 😺 como de error 😿) con el que nos aseguramos que realmente ha llegado algún evento y efectivamente se ha llamado al subscriber. En el mismo viaje comprobaremos en el callable que el evento que deserializamos es el mismo que hemos enviado antes 👍 y no se está perdiendo nada al pasar el Json a `DomainEvent`

Por otro lado, lo que hará el test será crearnos un _fakeSubscriber_ en el que hemos definido que [se subscriba](https://github.com/CodelyTV/php-ddd-skeleton/blob/55a7764e2feb26e1966b63f8ffd8e3b9e3f6428f/tests/src/Shared/Infrastructure/Bus/Event/RabbitMq/TestAllWorksOnRabbitMqEventsPublished.php#L13-L19) a dos eventos de dominio y que utilizaremos tanto para generar el nombre formateado de la cola como para probar que el evento publicado se termina consumiendo correctamente

¿Alguna Duda?
=============

Si tienes alguna duda sobre este video o quieres compartir alguna sugerencia no dudes en abrir una nueva discusión más abajo 👇👇👇

¡Nos vemos en el siguiente video: 🙅🏾‍♂️ Gestión de errores al consumir con RabbitMQ: Colas de retry y dead letter!