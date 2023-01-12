♻️ … 🚌💨 Integración y refactor a QueryBus
===========================================

Cerramos esta lección integrando el QueryBus que tal como vamos a ver no implica muchas diferencias con respecto a la integración del CommandBus que acabamos de ver. En concreto la diferencia radica en que ahora esperaremos que el **QueryBus pueda devolver una respuesta** si encuentra resultados

Interface `QueryBus`:

    interface QueryBus
    {
        public function ask(Query $query): ?Response;
    }


Esta respuesta será una implementación de `Response` que consiste en un DTO con los datos en plano que devolveremos al controlador

Integrando el QueryBus 🚌
-------------------------

La Implementación ‘en memoria’ del `QueryBus` es absolutamente similar a la del `CommandBus` (volveremos a aprovechar la mágia de Symfony para registrar y mapear los QueryHandlers)

Es en el método _ask()_ donde encontraremos diferencias, ya que en este caso si que queremos devolver la respuesta del caso de uso (Ojo 👀 tal como funciona Symfony tendremos que [coger la última respuesta](https://github.com/CodelyTV/php-ddd-skeleton/blob/d67e903ddbc3f01d888bd636ea715387667aea32/src/Shared/Infrastructure/Bus/Query/InMemorySymfonyQueryBus.php#L36) y extraer el `result` que contiene dentro)

Desde el controlador también tendremos que cambiar el constructor para que reciba el QueryBus en lugar del propio Caso de uso y, en el _\_\_invoke()_ llamaremos al método _ask()_ pasándole la query en lugar de la request

Clase `FindCoursesCounterQueryHandler`:

    final class FindCoursesCounterQueryHandler implements QueryHandler
    {
        private $finder;
    
        public function __construct(CoursesCounterFinder $finder)
        {
            $this->finder = $finder;
        }
        
        public function __invoke(FindCoursesCounterQuery $query): CoursesCounterResponse
        {
            return $this->finder->__invoke();
        }
    }


Será el QueryHandler el encargado de transformar los datos de la query (si los hubiera) a Value Objects y pasárselos al caso de uso al ser invocado

Adaptando los tests unitarios ✅
-------------------------------

Tal como hicimos con el CommandBus, tendremos que adaptar nuestra suite de tests unitarios, instanciando el `QueryHandler` en lugar del caso de uso

Test `FindCoursesCounterQueryHandlerTest`:

    final class FindCoursesCounterQueryHandlerTest extends CoursesCounterModuleUnitTestCase
    {
        private $handler;
    
        protected function setUp(): void
        {
            parent::setUp();
            $this->handler = new FindCoursesCounterQueryHandler(new CoursesCounterFinder($this->repository()));
        }
        
        /** @test */
        public function it_should_find_an_existing_courses_counter(): void
        {
            $counter  = CoursesCounterMother::random();
            $query    = new FindCoursesCounterQuery();
            $response = CoursesCounterResponseMother::create($counter->total());
            $this->shouldSearch($counter);
            $this->assertAskResponse($response, $query, $this->handler);
        }
        
        /** @test */
        public function it_should_throw_an_exception_when_courses_counter_does_not_exists(): void
        {
            $query = new FindCoursesCounterQuery();
            $this->shouldSearch(null);
            $this->assertAskThrowsException(CoursesCounterNotExist::class, $query, $this->handler);
        }
    }


Al no ser una query a la que le tengamos que inyectar parámetros, no resulta necesario crear un Object Mother para los tests y, en su lugar podemos utilizar directamente la query

¿Alguna Duda?
=============

Si tienes alguna duda sobre este video o quieres compartir alguna sugerencia no dudes en abrir una nueva discusión más abajo 👇👇👇

¡Nos vemos en la siguiente lección: 🥡 Aplicaciones, Bounded Contexts y módulos!