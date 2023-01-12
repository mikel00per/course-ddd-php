👂 Subscriber a todos los eventos
=================================

Durante el curso hemos trabajado con un caso en el que los Subscriptor sólo escuchaban ante un Evento de Dominio 🙋‍♂️, pero este no dejaría de ser un caso bastante sencillo ¿Y si quisieramos que un Subscriptor atendiera a todos los eventos 👨‍👩‍👧‍👦?

Si recordamos la definición de los subscribers, el hecho de extender de `DomainEventSubscriber` los obliga a implementar el método _subscribedTo()_ en el cual especificamos en un array aquellos eventos a los que se subscribe. En caso de que quisieramos añadir por ejemplo un segundo evento para que ejecute el mismo caso de uso, lo incluiríamos en este array y modificaríamos el método _\_\_invoke()_ eliminando el Type Hinting del argumento y especificando en el comentario las clases que espera recibir. Ojo 👀, en este punto tendremos que controlar qué clase es la que llega en caso de hacer llamadas a métodos que sólo tenga una de ellas

¿Y si queremos que se subscriba a todos los eventos? Es un riesgo modificar estos métodos a la hora de incluir todos nuestros eventos de dominio, puesto que podríamos dejar atrás cualquier detalle. Como alternativa lo que podemos hacer es indicar que en lugar de subscribirse a los N eventos, lo haga directamente a todos los `DomainEvent` (Haciendo uso de la Herencia)

Clase `StoreDomainEventOnOccurred`:

    final class StoreDomainEventOnOccurred implements DomainEventSubscriber
    {
        private $storer;
    
        public function __construct(DomainEventStorer $storer)
        {
            $this->storer = $storer;
        }
    
        public static function subscribedTo(): array
        {
            return [DomainEvent::class];
        }
    
        public function __invoke(DomainEvent $event)
        {
            $id          = new AnalyticsDomainEventId($event->eventId());
            $aggregateId = new AnalyticsDomainEventAggregateId($event->aggregateId());
            $name        = new AnalyticsDomainEventName($event::eventName());
            $body        = new AnalyticsDomainEventBody($event->toPrimitives());
            $this->storer->store($id, $aggregateId, $name, $body);
        }
    }


Esta configuración va a resultar muy util por ejemplo a la hora de llevar a cabo analíticas acerca de los eventos que se producen en nuestro sistema. En este supuesto vemos cómo el caso de uso recupera tanto los datos genéricos del evento como el body (gracias al método abstracto _toPrimitives()_ que implementan todos los eventos) para serializarlo y guardarlo en BD de cara a futuras explotaciones (Recordad que en el [curso de Elasticsearch](https://pro.codely.tv/library/elkbeats-centraliza-la-gestion-de-logs-con-the-elastic-stack/about/) podéis ver con más detalle cómo recoger y explotar datos para este tipo de analíticas)

Finalmente, será necesario configurar cada implementación de `EventBus` que tengamos para que acepte esta lógica (En el caso del Bus Síncrono tenemos la suerte de que Symfony Messenger se ocupa de hacer la magia por nosotros)

¿Alguna Duda?
=============

Si tienes alguna duda sobre este video o quieres compartir alguna sugerencia no dudes en abrir una nueva discusión más abajo 👇👇👇

¡Nos vemos en el siguiente Video: 🤕 Gestión de errores!