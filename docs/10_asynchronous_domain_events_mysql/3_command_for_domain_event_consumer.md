🤖 Comando para el consumidor de eventos de dominio
===================================================

Siguiendo el enfoque de gestionar nuestros Eventos de Dominio a través de MySQL sólo nos queda plantear cómo los consumiremos, para ellos utilizaremos un comando que podremos ejecutar periódicamente desde un cron. Al margen de cómo ejecutarlo vamos a entrar a ver que hará internamente

Clase `ConsumeMySqlDomainEventsCommand`:

    final class ConsumeMySqlDomainEventsCommand extends Command
    {
        protected static $defaultName = 'codelytv:domain-events:mysql:consume';
        private $consumer;
        private $subscriberLocator;
        private $connections;
    
        public function __construct(
            MySqlDoctrineDomainEventsConsumer $consumer,
            DatabaseConnections $connections,
            DomainEventSubscriberLocator $subscriberLocator
        ) {
            $this->consumer          = $consumer;
            $this->subscriberLocator = $subscriberLocator;
            $this->connections       = $connections;
            parent::__construct();
        }
    
        protected function configure(): void
        {
            $this
                ->setDescription('Consume domain events from MySql')
                ->addArgument('quantity', InputArgument::REQUIRED, 'Quantity of events to process');
        }
    
        protected function execute(InputInterface $input, OutputInterface $output)
        {
            $quantityEventsToProcess = (int) $input->getArgument('quantity');
            $consumer = pipe($this->consumer(), $this->clearConnections());
            $this->consumer->consume($consumer, $quantityEventsToProcess);
        }
    
        private function consumer(): callable
        {
            return function (DomainEvent $domainEvent) {
                $subscribers = $this->subscriberLocator->allSubscribedTo(get_class($domainEvent));
                foreach ($subscribers as $subscriber) {
                    $subscriber($domainEvent);
                }
            };
        }
    
        private function clearConnections(): callable
        {
            return function () {
                $this->connections->clear();
            };
        }
    }


Debido a que en PHP la gestión de la memoria no es especialmente óptima 🤔, es importante que definamos una cantidad relativamente reducida de eventos a consumir en cada proceso (es recomendable que muera antes y volver a levantarlo las veces necesarias a mantenerlo abierto incrementando mas y mas el consumo de memoria y haciendo que cada vez vaya mas lento)

Vemos dentro de la función _execute()_ cómo estamos llamando a _subscribers()_ para montar un callable que pasaremos como argumento a nuestro `MySqlDoctrineDomainEventsConsumer`. Podemos entender este callable como un subscriptor que va a llamar a todos los subscriptores de nuestros eventos 👨‍👧‍👦

Una vez recogidos todos los subscriptores los iteraremos y ejecutaremos con el evento de dominio que estamos recibiendo (gracias a que tenemos definidos en ellos el método _invoke()_)

Finalmente quien ejecutará esta lógica será el consumidor, y cada vez que se ejecuten llamaremos a _cleanConnections()_ para que limpie la BD de conexiones (evitando problemas en condiciones de carreras con nuestra BD)

Cuando analizamos el módo en que se llaman y ejecutan los subscriptores nos puede chirriar el hecho de no estar controlando que si uno de estos subscriptores falla, se estará produciendo un error. Aunque es un problema que hemos contemplado, lo que haremos será irnos directamente a ver la propuesta a través de un sistema de colas como RabbitMQ 🐇

¿Alguna Duda?
=============

Si tienes alguna duda sobre este video o quieres compartir alguna sugerencia no dudes en abrir una nueva discusión más abajo 👇👇👇

¡Nos vemos en la siguiente lección: ⚡️ Publicar eventos de dominio asíncronos y escalables utilizando RabbitMQ!