Idempotencia al consumir eventos
================================

Veíamos el tema de la idempotencia en el test de aceptación creado para el caso de uso de incrementar el contador de cursos. Lo que refleja el test y que debemos controlar en nuestro código es la posibilidad de que el Bus envíe un evento duplicado 🔀

*   Podéis encontrar el código relativo a este video en el tag [0.17.0 del repo](https://github.com/CodelyTV/php-ddd-skeleton/tree/0.17.0)

Definiendo el Subscriber 📢
---------------------------

Clase `IncrementCoursesCounterOnCourseCreated`:

    final class IncrementCoursesCounterOnCourseCreated implements DomainEventSubscriber
    {
        private $incrementer;
        public function __construct(CoursesCounterIncrementer $incrementer)
        {
            $this->incrementer = $incrementer;
        }
        public static function subscribedTo(): array
        {
            return [CourseCreatedDomainEvent::class];
        }
        public function __invoke(CourseCreatedDomainEvent $event): void
        {
            $courseId = new CourseId($event->aggregateId());
            apply($this->incrementer, [$courseId]);
        }
    }


Podemos ver cómo el método _invoke()_ del subscriber está recibiendo por parámetro un `CourseCreatedDomainEvent` (recordar que es el EventBus quien estará llamando a este método) del que recuperaremos el id del curso que se ha creado, lo que haremos con este id será pasárselo por constructor al caso de uso `CoursesCounterIncrementer`

Definiendo el Caso de Uso 📈
----------------------------

Clase `CoursesCounterIncrementer`:

    final class CoursesCounterIncrementer
    {
        private $repository;
        private $uuidGenerator;
        private $publisher;
    
        public function __construct(
            CoursesCounterRepository $repository,
            UuidGenerator $uuidGenerator,
            DomainEventPublisher $publisher
        ) {
            $this->repository    = $repository;
            $this->uuidGenerator = $uuidGenerator;
            $this->publisher     = $publisher;
        }
        
        public function __invoke(CourseId $courseId)
        {
            $counter = $this->repository->search() ?: $this->initializeCounter();
            if (!$counter->hasIncremented($courseId)) {
                $counter->increment($courseId);
                $this->repository->save($counter);
                $this->publisher->publish(...$counter->pullDomainEvents());
            }
        }
        
        private function initializeCounter(): CoursesCounter
        {
            return CoursesCounter::initialize(new CoursesCounterId($this->uuidGenerator->generate()));
        }
    }


Lo primero que haremos cuando se llame al caso de uso será comprobar si ya tenemos un contador creado o si por el contrario debemos ‘inicializarlo’. Este contador corresponde con el agregado `CoursesCounter`

Una pieza importante dentro de este agregado es que guardaremos en un atributo de tipo array los ids de los cursos que ya han sido procesados, lo cual nos permitirá ignorar aquellas peticiones que recibamos por eventos duplicados (Ojo! 👀 que no estaremos lanzándo ningún error, simplemente no llevaremos a cabo la lógica del caso de uso). En el caso de no haber sido procesado llamaremos al método _increment()_ del agregado, que será el encargado de incrementar internamente el valor del contador y añadir el `courseId` al array que comentábamos

*   La opción de un array de ids es útil con un número reducido de elementos, no obstante, con un número elevado podría interesarnos más delegar esa gestión a un almacenamiento en Redis, por ejemplo

En los [tests para este caso de uso](https://github.com/CodelyTV/php-ddd-skeleton/blob/0.17.0/tests/src/Mooc/CoursesCounter/Application/Increment/IncrementCoursesCounterOnCourseCreatedTest.php) debemos tener en cuenta que en la comunicación con los distintos colaboradores estaremos asumiendo que se están llamando con los parámetros que deberíamos (veremos más adelante cómo realizar estas validaciones de forma sencilla)

El punto de partida de cada test será la generación del evento de dominio, y este evento será lanzado directamente al subscriber (sin que pase por el Bus), que lo inicializaremos en el _setUp()_ de esta suite de tests

¿Alguna Duda?
=============

Si tienes alguna duda sobre este video o quieres compartir alguna sugerencia no dudes en abrir una nueva discusión más abajo 👇👇👇

¡Nos vemos en la siguiente lección: 🤷‍♀️‍ Testeando “parcialmente”: PhpUnit SimilarComprator!