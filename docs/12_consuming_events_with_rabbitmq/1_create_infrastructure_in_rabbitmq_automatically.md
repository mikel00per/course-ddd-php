🎱 Crear infraestructura en RabbitMQ automáticamente: CI pipelines FTW
======================================================================

### Configuración y Creación del Exchange 🔶

Vamos a ver cómo podemos automatizar el proceso de creación de nuestro exchange y las diferentes colas que nos permitan dirigir los eventos a todos sus subscriptores tal y como ya hemos visto

Configuración de RabbitMQ en el `docker-compose`:

      rabbitmq:
        container_name: codelytv-php_ddd_skeleton-rabbitmq
        image: 'rabbitmq:3.7-management'
        restart: unless-stopped
        ports:
          - 5630:5672
          - 8090:15672
        environment:
          - RABBITMQ_DEFAULT_USER=codelytv
          - RABBITMQ_DEFAULT_PASS=c0d3ly


En primer lugar vemos la configuración que utilizaremos para levantar RabbitMQ con su panel de administración web (para lo cual le estamos indicando el _\-management_ en la imagen), especificando tanto los puertos para publicación y acceso al panel web como las variables de entorno necesarias

*   Ojo 👀, Existe un bug en Docker para Mac que nos lanzará un error si usamos un puerto >15000 en local, por lo que tendremos que ‘mimificarlo’ a un número menor (el que utiliza RabbitMQ es el 15672)

Clase `RabbitMqConfigurer` (Resultado final):

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


Desde el método configure lo que haremos simplemente será la **declaración del exchange y de las colas** (En el código del repositorio podéis ver que se han añadido además las colas de _retry_ y _deadLetter_ ). Es importante que el `exchangeName` lo reciba precisamente el _configure()_ y no el constructor porque esto nos permitirá utilizar la misma instancia y ejecutar este método por cada Bounded Context

Al declarar un nuevo **exchange** le estaremos indicando que sea de tipo Topic (como comentábamos en la lección anterior) y que además sea durable, es decir, no queremos que se borre salvo que nosotros se lo indiquemos. La ventaja que nos ofrece utilizar la librería de bajo nivel de AMQP es que si intentamos crear un exchange o una cola ya existentes no nos generará un error, simplemente no hará nada

### Declarando colas 🐇

Clase `RabbitMqConfigurer` (Resultado final):

    final class RabbitMqConfigurer
    {
        // ,,,
    
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


En el caso de la declaración de colas, lo primero que haremos por cada una de ellas será definir el nombre utilizando la clase [RabbitMqQueueNameFormatter](https://github.com/CodelyTV/php-ddd-skeleton/blob/master/src/Shared/Infrastructure/Bus/Event/RabbitMq/RabbitMqQueueNameFormatter.php) que hemos creado para tener un ‘estándar’ en la nomenclatura

Al igual que en el exchange, declararemos que las colas sean durables (dado el uso que queremos darle a RabbitMQ, no nos interesa que sean elementos temporales que se borren lejos de nuestro control). Una vez declarada, debemos bindear cada cola a los eventos de dominio

Una vez tenemos toda la configuración lista, podemos ejecutarlo lanzando un [comando](https://github.com/CodelyTV/php-ddd-skeleton/blob/master/apps/mooc/backend/src/Command/DomainEvents/RabbitMq/ConfigureRabbitMqCommand.php) de symfony desde la consola:

    php apps/mooc/backend/bin/console codelytv:domain-events:rabbitmq:configure


Añadiremos este comando en nuestra pipeline de despliegue para que se ejecute cada vez que hagamos deploy 🚀

Podéis ver el test de esta pieza de la infraestructura [aquí](https://github.com/CodelyTV/php-ddd-skeleton/blob/master/tests/src/Shared/Infrastructure/Bus/Event/RabbitMq/RabbitMqEventBusTest.php), donde lo que vamos a comprobar es jústamente la infraestructura real y podremos verlo directamente en el panel web de RabbitMQ (Por ello también borraremos estos valores antes de cada ejecución de los tests)

¿Alguna Duda?
=============

Si tienes alguna duda sobre este video o quieres compartir alguna sugerencia no dudes en abrir una nueva discusión más abajo 👇👇👇

¡Nos vemos en el siguiente video: 🌬 Consumir eventos desde RabbitMQ!