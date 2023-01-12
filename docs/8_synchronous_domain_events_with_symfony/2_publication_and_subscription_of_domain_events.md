Publicación y suscripción de eventos de dominio y test de aceptación
====================================================================

Vamos a ver cómo se guardan los Eventos de Dominio en el AggregateRoot, además de las implicaciones que tiene la publicación de eventos en nuestros tests y cómo podemos validarlo

*   Podéis encontrar el código relativo a este video en el tag [0.16.0 del repo](https://github.com/CodelyTV/php-ddd-skeleton/tree/0.16.0)

Publicación y subscripción de eventos de dominio 📥
---------------------------------------------------

Clase `CourseCreator`:

    final class CourseCreator
    {
        private $repository;
        private $bus;
    
        public function __construct(CourseRepository $repository, EventBus $bus)
        {
            $this->repository = $repository;
            $this->bus        = $bus;
        }
        
        public function __invoke(CourseId $id, CourseName $name, CourseDuration $duration)
        {
            $course = Course::create($id, $name, $duration);
        
            $this->repository->save($course);
            
            $this->bus->publish(...$course->pullDomainEvents());
        }
    }


Estamos pasando como parámetro en el constructor el EventBus y, gracias al autowiring de Symfony y que no tenemos más implementaciones, no necesitaremos mayor configuración para que nos inyecte nuestro `InMemorySymfonyEventBus`. Una vez se registre el evento de dominio dentro del AggregateRoot `Course` y lo hayamos persistido en BD, llamaremos al método _publish()_ del EventBus para que publique todos los eventos del agregado.

Clase `CourseCreateDomainEvent`:

    final class CourseCreatedDomainEvent extends DomainEvent
    {
        private $name;
        private $duration;
        public function __construct(
            string $id,
            string $name,
            string $duration,
            string $eventId = null,
            string $occurredOn = null
        ) {
            parent::__construct($id, $eventId, $occurredOn);
            $this->name     = $name;
            $this->duration = $duration;
        }
    
        public static function eventName(): string
        {
            return 'course.created';
        }
    
        public function toPrimitives(): array
        {
            return [
                'name'     => $this->name,
                'duration' => $this->duration,
            ];
        }
    
        public static function fromPrimitives(
            string $aggregateId,
            array $body,
            string $eventId,
            string $occurredOn
        ): DomainEvent {
            return new self($aggregateId, $body['name'], $body['duration'], $eventId, $occurredOn);
        }
    }


Es importante que todos los atributos de nuestros eventos de dominio sean escalares, de modo que podamos serializarlo sin problema. Tanto `eventId` como `occurredOn` son atributos identificativos del propio evento que nos permiten tratarlo de forma única y saber exáctamente en qué momento se ha creado Por otra parte, también recogeremos siempre todos los atributos del agregado sobre el que se registra el evento Todos nuestros eventos de dominio van a implementar también tres métodos:

*   eventName: Donde definimos el nombre del evento
*   toPrimitives: En el que pasaremos a primitivos el body del evento (todos los atributos no identificativos)
*   fromPrimitives: Se ejecuta al recuperar un evento de dominio para mapearlo con los atributos recibidos

Clase `AggregateRoot`:

    abstract class AggregateRoot
    {
        private $domainEvents = [];
        
        final public function pullDomainEvents(): array
        {
            $domainEvents       = $this->domainEvents;
            $this->domainEvents = [];
            return $domainEvents;
        }
    
        final protected function record(DomainEvent $domainEvent): void
        {
            $this->domainEvents[] = $domainEvent;
        }
    }


De momento almacenamos estos eventos en un array en memoria dentro del propio agregado y que le llegaría heredado de `AggregateRoot`. Así podremos acceder a ellos en el caso de uso y publicarlos una vez se haya finalizado

Testeando nuestro EventBus Síncrono ✅
-------------------------------------

En el test de aceptación para el caso de uso de crear un nuevo curso no tendremos ningún cambio tras haber añadido la publicación de eventos de dominio ya que en nuestra opinión lo único que debemos comprobar en este caso es que nos devuelva un 201, sin entrar en que se guarde correctamente en BD o que se publique el evento correspondiente

Clase `CourseCreatorTest`:

    final class CourseCreatorTest extends CoursesModuleUnitTestCase
    {
        private $creator;
    
        protected function setUp(): void
        {
            parent::setUp();
            $this->creator = new CourseCreator($this->repository(), $this->domainEventPublisher());
        }
    
        /** @test */
        public function it_should_create_a_valid_course(): void
        {
            $request = CreateCourseRequestMother::random();
            $course      = CourseMother::fromRequest($request);
            $domainEvent = CourseCreatedDomainEventMother::fromCourse($course);
            $this->shouldSave($course);
            $this->shouldPublishDomainEvent($domainEvent);
            $this->creator->__invoke($request);
        }
    }


Por otra parte, en el caso de `CourseCreatorTest` si que comprobaremos que se haya publicado el evento. Necesitaremos añadir un mock de nuestro eventBus en la instanciación del caso de uso tal como hicimos con el repositorio

Finalmente, también añadiremos un nuevo [test de aceptación](https://github.com/CodelyTV/php-ddd-skeleton/blob/0.18.0/tests/apps/mooc/backend/features/courses_counter/courses_counter_get.feature) de cara a la implementación del subscribers que incremente el número de cursos publicados (dentro del módulo CourseCounter). En este test comprobaremos por un lado que al recibir un evento responda correctamente e incremente el contador de total de cursos, pero también validaremos que no incremente este total cuando recibamos eventos duplicados

¿Alguna Duda?
=============

Si tienes alguna duda sobre este video o quieres compartir alguna sugerencia no dudes en abrir una nueva discusión más abajo 👇👇👇

¡Nos vemos en el siguiente video: 🤟 Idempotencia al consumir eventos: Implementación del caso de uso y test unitario!