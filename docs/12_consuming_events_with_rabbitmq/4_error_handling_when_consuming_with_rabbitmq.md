🙅🏾‍♂️ Gestión de errores al consumir con RabbitMQ: Colas de retry y dead letter
=================================================================================

Terminamos el bloque con la gestión de errores en la consumición de Errores (Recomendamos encarecidamente hacer un alto y echarle un ojo 👀 a los videos relacionados con la gestión de colas en el curso de [Comunicación entre microservicios](https://pro.codely.tv/library/comunicacion-entre-microservicios-event-driven-architecture/74823/about/)) viendo cómo podemos llevarla a cabo cuando trabajamos con RabbitMQ

Planteando la gestión de errores
--------------------------------

En esta gestión entran en juego dos conceptos nuevos que van a facilitarnos la vida una vez implementados:

*   **Cola de Retry**: A esta cola llegarán los eventos que por algún error no se han procesado en la cola ‘padre’ para volver a enviarlos a ésta pasado un tiempo de demora
*   **Cola de Dead Letter**: Si tras N reintentos desde la cola de Retry no se ha procesado el evento, lo enviaremos a ésta cola, donde podremos configurar una alarma que nos avise para intentar ejecutar el proceso a mano y localizar dónde está el problema

El planteamiento será en primer lugar crear dos nuevos exchanges (retry y dead-letter) puesto que utilizaremos una binding key nueva que, en lugar de corresponder con el nombre del evento, lo hará con el nombre de la cola original para que cuando volvamos a re-encolar el evento lo hagamos en esa cola original y no a cada uno de los consumers que haya

Cada uno de estos dos exchanges tendrá a su vez las mismas colas que el exchange principal (pero con distinta binding key como explicábamos)

Con esta estructura, el procedimiento será, al producirse un error al consumir el evento, enviar éste a la cola de ‘retry’ incrementando el contador de reintentos que tendremos en la metadata y seguidamente hacerle ack en la cola ‘padre’ Desde retry se volverá a lanzar el evento al exchange principal tantas veces como le especifiquemos (nosotros lo hemos acotado a 5 reintentos) tras lo cual, si sigue devolviéndonos un error, pasará a enviarse a la cola de ‘dead letter’ en la que se quedará sin hacer nada más

Creando los exchanges de retry y dead letter
--------------------------------------------

Clase `RabbitMqConfigurer` (Declaración de los exchanges):

    final class RabbitMqConfigurer
    {
        private $connection;
    
        public function __construct(RabbitMqConnection $connection)
        {
            $this->connection = $connection;
        }
    
        public function configure(string $exchangeName, DomainEventSubscriber ...$subscribers): void
        {
            $retryExchangeName      = RabbitMqExchangeNameFormatter::retry($exchangeName);
            $deadLetterExchangeName = RabbitMqExchangeNameFormatter::deadLetter($exchangeName);
            $this->declareExchange($exchangeName);
            $this->declareExchange($retryExchangeName);
            $this->declareExchange($deadLetterExchangeName);
            $this->declareQueues($exchangeName, $retryExchangeName, $deadLetterExchangeName, ...$subscribers);
        }
    
        private function declareExchange(string $exchangeName): void
        {
            $exchange = $this->connection->exchange($exchangeName);
            $exchange->setType(AMQP_EX_TYPE_TOPIC);
            $exchange->setFlags(AMQP_DURABLE);
            $exchange->declareExchange();
        }
    
      // ...
    }


Hemos modificado nuestra clase de configuración para que además de declarar el exchange padre, declare los exchanges de retry y dead letter, así como las colas para cada uno de ellos. Como se ve en el método _configure()_, llamamos las tres veces a _declareExchange()_ pero previamente estaremos formateando el nombre de los exchange de retry y dead letter añadiendoles el prefijo correspondiente

Clase `RabbitMqConfigurer` (Declaración de las colas):

    final class RabbitMqConfigurer
    {
        
        // ...
    
        private function declareQueues(
            string $exchangeName,
            string $retryExchangeName,
            string $deadLetterExchangeName,
            DomainEventSubscriber ...$subscribers
        ): void {
            each($this->queueDeclarator($exchangeName, $retryExchangeName, $deadLetterExchangeName), $subscribers);
        }
        private function queueDeclarator(
            string $exchangeName,
            string $retryExchangeName,
            string $deadLetterExchangeName
        ): callable {
            return function (DomainEventSubscriber $subscriber) use (
                $exchangeName,
                $retryExchangeName,
                $deadLetterExchangeName
            ) {
                $queueName           = RabbitMqQueueNameFormatter::format($subscriber);
                $retryQueueName      = RabbitMqQueueNameFormatter::formatRetry($subscriber);
                $deadLetterQueueName = RabbitMqQueueNameFormatter::formatDeadLetter($subscriber);
                $queue           = $this->declareQueue($queueName);
                $retryQueue      = $this->declareQueue($retryQueueName, $exchangeName, $queueName, 1000);
                $deadLetterQueue = $this->declareQueue($deadLetterQueueName);
                $queue->bind($exchangeName, $queueName);
                $retryQueue->bind($retryExchangeName, $queueName);
                $deadLetterQueue->bind($deadLetterExchangeName, $queueName);
                foreach ($subscriber::subscribedTo() as $eventClass) {
                    $queue->bind($exchangeName, $eventClass::eventName());
                }
            };
        }
    
        private function declareQueue(
            string $name,
            string $deadLetterExchange = null,
            string $deadLetterRoutingKey = null,
            int $messageTtl = null
        ): AMQPQueue {
            $queue = $this->connection->queue($name);
            if (null !== $deadLetterExchange) {
                $queue->setArgument('x-dead-letter-exchange', $deadLetterExchange);
            }
            if (null !== $deadLetterRoutingKey) {
                $queue->setArgument('x-dead-letter-routing-key', $deadLetterRoutingKey);
            }
            if (null !== $messageTtl) {
                $queue->setArgument('x-message-ttl', $messageTtl);
            }
            $queue->setFlags(AMQP_DURABLE);
            $queue->declareQueue();
            return $queue;
        }
    }


A la hora de configurar las colas, no solo declararemos las correspondientes a cada exchange sino que también las estamos bindeando con los eventos de dominio y el nombre de la cola (que nos permitirá devolver como hemos visto los eventos a la cola del exchange padre). En el caso de la cola de retry, añadiremos además los parámetros necesarios para indicarle que republique los eventos que le lleguen pasado 1 segundo en el exchange principal

Manejando errores al consumir eventos
-------------------------------------

Clase `RabbitMqDomainEventsConsumer`:

    final class RabbitMqDomainEventsConsumer
    {
    
        // ...
    
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
    
        private function handleConsumptionError(AMQPEnvelope $envelope, AMQPQueue $queue): void
        {
            $this->hasBeenRedeliveredTooMuch($envelope)
                ? $this->sendToDeadLetter($envelope, $queue)
                : $this->sendToRetry($envelope, $queue);
            $queue->ack($envelope->getDeliveryTag());
        }
    
        private function hasBeenRedeliveredTooMuch(AMQPEnvelope $envelope): bool
        {
            return get('redelivery_count', $envelope->getHeaders(), 0) >= $this->maxRetries;
        }
    
        private function sendToDeadLetter(AMQPEnvelope $envelope, AMQPQueue $queue): void
        {
            $this->sendMessageTo(RabbitMqExchangeNameFormatter::deadLetter($this->exchangeName), $envelope, $queue);
        }
    
        private function sendToRetry(AMQPEnvelope $envelope, AMQPQueue $queue): void
        {
            $this->sendMessageTo(RabbitMqExchangeNameFormatter::retry($this->exchangeName), $envelope, $queue);
        }
    
        private function sendMessageTo(string $exchangeName, AMQPEnvelope $envelope, AMQPQueue $queue): void
        {
            $headers = $envelope->getHeaders();
            $this->connection->exchange($exchangeName)->publish(
                $envelope->getBody(),
                $queue->getName(),
                AMQP_NOPARAM,
                [
                    'message_id'       => $envelope->getMessageId(),
                    'content_type'     => $envelope->getContentType(),
                    'content_encoding' => $envelope->getContentEncoding(),
                    'priority'         => $envelope->getPriority(),
                    'headers'          => assoc($headers, 'redelivery_count', get('redelivery_count', $headers, 0) + 1),
                ]
            );
        }
    }


Ahora podemos modificar el consumidor de modo que cuando capturemos un error (cualquier Throwable) llamaremos a _handleConsumptionError()_, donde una vez decidido a qué cola enviar el evento le haremos el ack en la cola ‘padre’. Para ver si se ha enviado demasiadas veces a retry lo que haremos será consultar el contador que, como habíamos visto anteriormente, iría en el header del envoltorio que RabbitMQ coloca sobre el Json del evento Es importante tener en cuenta que cada vez que reenviamos el evento lo enviaremos con los mismos datos de la cabecera pero incrementando el contador _redelivery\_count_

Tests
-----

Test `RabbitMqEventBusTest`:

    final class RabbitMqEventBusTest extends InfrastructureTestCase
    {
    
        // ...
    
        /** @test */
        public function it_should_throw_an_exception_consuming_non_existing_domain_events(): void
        {
            $this->expectException(RuntimeException::class);
            $domainEvent = CoursesCounterIncrementedDomainEventMother::random();
            $this->configurer->configure($this->exchangeName, $this->fakeSubscriber);
            $this->publisher->publish($domainEvent);
            $this->consumer->consume(
                $this->assertConsumer($domainEvent),
                RabbitMqQueueNameFormatter::format($this->fakeSubscriber)
            );
            $this->assertTrue($this->consumerHasBeenExecuted);
        }
        /** @test */
        public function it_should_retry_failed_domain_events(): void
        {
            $domainEvent = CourseCreatedDomainEventMother::random();
            $this->configurer->configure($this->exchangeName, $this->fakeSubscriber);
            $this->publisher->publish($domainEvent);
            $this->simulateErrorConsuming();
            sleep(1);
            $this->consumer->consume(
                $this->assertConsumer($domainEvent),
                RabbitMqQueueNameFormatter::format($this->fakeSubscriber)
            );
            $this->assertTrue($this->consumerHasBeenExecuted);
        }
        /** @test */
        public function it_should_send_events_to_dead_letter_after_retry_failed_domain_events(): void
        {
            $domainEvent = CourseCreatedDomainEventMother::random();
            $this->configurer->configure($this->exchangeName, $this->fakeSubscriber);
            $this->publisher->publish($domainEvent);
            $this->simulateErrorConsuming();
            sleep(1);
            $this->simulateErrorConsuming();
            $this->assertDeadLetterContainsEvent(1);
        }
        private function assertConsumer(DomainEvent ...$expectedDomainEvents): callable
        {
            return function (DomainEvent $domainEvent) use ($expectedDomainEvents): void {
                $this->assertContainsEquals($domainEvent, $expectedDomainEvents);
                $this->consumerHasBeenExecuted = true;
            };
        }
        private function failingConsumer(): callable
        {
            return static function (DomainEvent $domainEvent): void {
                throw new RuntimeException('To test');
            };
        }
        private function simulateErrorConsuming(): void
        {
            try {
                $this->consumer->consume(
                    $this->failingConsumer(),
                    RabbitMqQueueNameFormatter::format($this->fakeSubscriber)
                );
            } catch (Throwable $error) {
                $this->assertInstanceOf(RuntimeException::class, $error);
            }
        }
        private function cleanEnvironment(RabbitMqConnection $connection): void
        {
            $connection->queue(RabbitMqQueueNameFormatter::format($this->fakeSubscriber))->delete();
            $connection->queue(RabbitMqQueueNameFormatter::formatRetry($this->fakeSubscriber))->delete();
            $connection->queue(RabbitMqQueueNameFormatter::formatDeadLetter($this->fakeSubscriber))->delete();
        }
        private function assertDeadLetterContainsEvent(int $expectedNumberOfEvents): void
        {
            $totalEventsInDeadLetter = 0;
            while ($this->connection->queue(RabbitMqQueueNameFormatter::formatDeadLetter($this->fakeSubscriber))->get(
                AMQP_AUTOACK
            )) {
                $totalEventsInDeadLetter++;
            }
            $this->assertSame($expectedNumberOfEvents, $totalEventsInDeadLetter);
        }
    }


El primer cambio que hemos añadido para estos tests es que ahora debemos asegurarnos de eliminar no sólo las colas de la rama principal sino también las de retry y dead\_letter antes de cada prueba

Para comprobar que realiza el retry correctamente lo que haremos será forzar un primer intento fallido (pasándole un _failingConsumer()_ al consumidor) y esperando un segundo (tiempo en que la cola de retry manda el evento a la cola principal) antes de volver a tratar de consumir el mismo evento

En el caso de la cola de dead letter el proceso será forzar de nuevo tantos intentos fallidos de consumir el evento como hayamos definido de máximo para, posteriormente, revisar que efectivamente encontramos ese mismo evento en la cola de dead letter

Estas modificaciones no nos repercutirán en el diseño de los tests de aceptación puesto que tal como explicábamos al inicio del curso, éstos se ejecutarán atacando al Bus local, priorizando que estos se ejecuten más rápido y sean los tests unitarios los que nos aseguren que todo va fino en la implementación de infraestructura

¿Alguna Duda?
=============

Si tienes alguna duda sobre este video o quieres compartir alguna sugerencia no dudes en abrir una nueva discusión más abajo 👇👇👇

¡Nos vemos en la siguiente lección: 🔀 CQRS!