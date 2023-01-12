♻️ Refactoring de Arquitectura Hexagonal a CQRS
===============================================

Ya que vamos a refactorizar nuestro código para dar el paso a CQRS, es fundamental que nuestro [test de aceptación](https://github.com/CodelyTV/php-ddd-skeleton/blob/0.18.0/tests/apps/mooc/backend/features/courses/course_put.feature) y la suite de [tests unitarios](https://github.com/CodelyTV/php-ddd-skeleton/blob/0.18.0/tests/src/Mooc/Courses/Application/Create/CourseCreatorTest.php) se lancen y nos aseguren que todo funciona correctamente

En este nivel de tests, una modificación que tendremos que acometer es que los **test unitarios ahora atacarán desde el CommandHandler** y no desde el Caso de uso

Refactorizando de Fuera hacia Dentro 📥
---------------------------------------

Mantenemos nuestra forma de trabajar Outside-In, así que el primer paso será modificar el Controller 📥, que ya no llamará directamente al Caso de uso sino que lanzará un comando al CommandBus

Clase `CoursesPutController`:

    final class CoursesPutController
    {
        private $bus;
        public function __construct(CommandBus $bus)
        {
            $this->bus = $bus;
        }
        public function __invoke(string $id, Request $request)
        {
            $this->bus->dispatch(
                new CreateCourseCommand(
                    $id,
                    $request->request->get('name'),
                    $request->request->get('duration')
                )
            );
            return new Response('', Response::HTTP_CREATED);
        }
    }


El hecho de venir trabajando con el objeto `Request` nos facilita enormemente esta tarea y los cambios son claramente mínimos 👌. Vemos cómo, aunque lo que manejamos ahora es un CommandBus en lugar del `CourseCreator`, pasándole un `CreateCourseCommand` en vez de una `CreateCourseRequest`, los parámetros que le estamos pasando son exáctamente los mismos (Al estar mimificando el mismo patrón, las modificaciones se reducen y simplifican enormemente 🤯)

Lo que si será necesario en este caso será que el nuevo comando que enviamos en el _dispatch()_ implemente la interface de `Command`, pues es lo que el CommandBus espera por contrato (en el caso de la Request no teníamos este compromiso por lo que no implementaba ninguna interface)

Clase `CreateCourseCommandHandler`:

    final class CreateCourseCommandHandler implements CommandHandler
    {
        private $creator;
        public function __construct(CourseCreator $creator)
        {
            $this->creator = $creator;
        }
        public function __invoke(CreateCourseCommand $command)
        {
            $id       = new CourseId($command->id());
            $name     = new CourseName($command->name());
            $duration = new CourseDuration($command->duration());
            $this->creator->__invoke($id, $name, $duration);
        }
    }


Por otro lado también crearemos la implementación de Commandhandler ligada al `CreateCourseCommand`, recibiendo por constructor el caso de uso `CourseCreator` al que llamará desde su propio _\_\_invoke()_. Es decir, este commandHandler tiene la responsabilidad de instanciar los Value Objects desde los primitivos que recibe en el comando y llamar con ellos al caso de uso (Ahora el Caso de Uso no recibe una Request que descomponer, sino los propios Value Objects dejándolo mucho más limpio)

Se puede ver como lo que estamos haciendo aquí es exáctamente lo mismo que desde el EventSubscriber, como ya veníamos hablando desde el principio, el hecho de que los casos de uso sean agnósticos nos va a permitir que puedan ser llamados desde diferentes puntos de entrada

Modificando los tests unitarios ✅
---------------------------------

Clase `CreateCourseCommandHandlerTest`:

    final class CreateCourseCommandHandlerTest extends CoursesModuleUnitTestCase
    {
        private $handler;
        protected function setUp(): void
        {
            parent::setUp();
            $this->handler = new CreateCourseCommandHandler(new CourseCreator($this->repository(), $this->eventBus()));
        }
        /** @test */
        public function it_should_create_a_valid_course(): void
        {
            $command = CreateCourseCommandMother::random();
            $course      = CourseMother::fromRequest($command);
            $domainEvent = CourseCreatedDomainEventMother::fromCourse($course);
            $this->shouldSave($course);
            $this->shouldPublishDomainEvent($domainEvent);
            $this->dispatch($command, $this->handler);
        }
    }


Como decíamos, los tests unitarios también tendrán que adaptarse para instanciar el `CreateCourseCommandHandler` en lugar del caso de uso `CourseCreator`. Construiremos el Command del mismo modo que anteriormente hacíamos con la Request y se lo pasaremos al CommandHandler al llamarlo

Por lo demás, seguirá siendo el mismo Curso el que creemos y publicaremos el mismo Evento de Dominio

¿Alguna Duda?
=============

Si tienes alguna duda sobre este video o quieres compartir alguna sugerencia no dudes en abrir una nueva discusión más abajo 👇👇👇

¡Nos vemos en el siguiente video: ♻️ … 🚌💨 Integración y refactor a QueryBus!