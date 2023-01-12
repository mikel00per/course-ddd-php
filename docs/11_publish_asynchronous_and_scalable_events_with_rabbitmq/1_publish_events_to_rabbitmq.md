🐰 Publicar eventos en RabbitMQ
===============================

Aunque seguimos con un planteamiento asíncrono en la publicación de eventos, iremos un pasito más allá en pro de permitir una **mayor disponibiidad** en nuestra aplicación que la que podríamos tener si siguieramos gestionando estas publicaciones con una BD como MySQL. El camino que hemos optado por tomar es implementar **RabbitMQ**, un _Message Broker_ que sirve como sistema de colas

*   Es altamente recomendable en este punto volver a ver el contenido del curso de [Comunicación entre \[Micro\]servicios](https://pro.codely.tv/library/comunicacion-entre-microservicios-event-driven-architecture/74823/about/) para ver todos los conceptos relacionados con la comunicación con colas

### Topología de nuestro sistema de colas 🗺️

*   [tryrabbitmq](http://tryrabbitmq.com/): Esta web nos permite diseñar cómodamente la topología de nuestra mensajería con RabbitMQ

![rabbitmq](https://cdn.filestackcontent.com/do5vUX3FRDKhru3qmePz)

Siguiendo con nuestro caso de uso, el _productor_ será el módulo de cursos, el cual producirá eventos que irán a un _exchange_ de [tipo topic](https://www.rabbitmq.com/tutorials/amqp-concepts.html#exchanges). Este exchange definirá el routing que tomarán los eventos, bindeando en este caso course\_created con la cola de increment\_courses\_counter\_on\_course\_created.

Vemos en este punto una importante diferencia entre la gestión en MySQL y RabbitMQ relacionada con la posterior gestión de errores

*   **MySQL**: Todos los eventos irían a una única tabla y un proceso posterior distribuiría los eventos en función de los casos de uso subscritos. Necesitaríamos un segundo proceso para llevar los eventos a las tablas correspondientes a cada caso de uso subscrito.
*   **RabbitMQ**: Cada cola se define por evento y subscriptor, por lo que todos los eventos que publique el productor se irán distribuyendo directamente por mediación del exchange

Al final del flujo encontramos el consumidor, que en este caso será un proceso de PHP para el caso de uso ‘increment\_courses\_counter\_on\_course\_created’ (tendremos un proceso para cada caso de uso subscrito). Veremos más adelante cómo optimizar la inserción de nuevos productores y consumidores en nuestro sistema de colas.

### Conociendo a nuestro productor 👨‍🌾

Tal como hicimos para MySQL, necesitamos implemenetar un `EventBus` para RabbitMQ dentro de nuestra infraestructura

Clase `RabbitMqEventBus`:

    final class RabbitMqEventBus implements EventBus
    {
        private $connection;
        private $exchangeName;
    
        public function __construct(
            RabbitMqConnection $connection,
            string $exchangeName,
        ) {
            $this->connection        = $connection;
            $this->exchangeName      = $exchangeName;
        }
    
        public function publish(DomainEvent ...$events): void
        {
            each($this->publisher(), $events);
        }
    
        private function publisher(): callable
        {
            return function (DomainEvent $event) {
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
            };
        }
    
    }


Un primer detalle importante de esta implementación es la `RabbitMqConnection` que recibe por constructor, esta conexión tira de una [librería de bajo nivel](https://pecl.php.net/package/amqp) de PECL

Gracias a esta librería de PECL podremos establecer una conexión persistente 💪 que levantaremos no por cada evento, sino por cada proceso/script

A la hora de publicar los eventos de dominio, lo haremos en el exchange (no sobre las colas directamente) que será quien conecte el evento (por la routingKey) a la cola (donde se define la bindingKey)

Por cada evento que nos llegue, lo que estaremos enviando será un json que contiene el evento serializado, la `routingKey` (nosotros hemos definido que será el nombre del evento) y el `messageId` que en nuestro caso corresponderá con el propio id del evento. Además será necesario especificar otros parámetros como el content type y el encoding de lo que estamos enviando

Como véis la publicación de eventos en RabbitMQ no esconde mucha mas magia, no obstante aún debemos trabajar en la gestión de errores en el momento de publicar ¡Y eso lo veremos en el próximo video!

¿Alguna Duda?
=============

Si tienes alguna duda sobre este video o quieres compartir alguna sugerencia no dudes en abrir una nueva discusión más abajo 👇👇👇

¡Nos vemos en el siguiente video: 🙆‍♂️ Error al publicar eventos: Fallback en MySql!