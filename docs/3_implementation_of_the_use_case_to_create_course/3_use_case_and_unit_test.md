Implementación del Caso de Uso y Test Unitario
==============================================

Vamos a implementar el caso de uso `CourseCreator`, donde recibiremos id, nombre y duración del curso y posteriormente se guardará en BD, aunque por el momento nos detendremos en la interface del repositorio

Es importante detenerse en el proceso de desarrollo, definir qué es lo que queremos y establecer desde un principio pruebas que nos permitan validar las implementaciones que finalmente piquemos, por eso recomendamos hacer el test antes que el código. Por otra parte, aún en el caso de que no hicieramos los tests antes que la implementación, si que deberíamos forzarnos a llevar el desarrollo de fuera hacia dentro (Outside-In) para no contaminar el diseño en base a la infraestructura que finalmente utilicemos 😷. Por eso resulta un aspecto clave definir primero el contrato con el cliente (Además llevará a que todo nuestro dominio y las acciones a realizar salgan de manera natural)

Clase `CoursesPutController`:

    final class CoursesPutController
    {
        private $creator;
    
        public function __construct(CourseCreator $creator)
        {
            $this->creator = $creator;
        }
    
        public function __invoke(string $id, Request $request)
        {
            $name     = $request->request->get('name');
            $duration = $request->request->get('duration');
    
            $this->creator->__invoke($id, $name, $duration);
    
            return new Response('', Response::HTTP_CREATED);
        }
    }


Ya en el video anterior comentamos que [CoursesPutController](https://github.com/CodelyTV/php-ddd-skeleton/blob/0.8.0/apps/mooc/backend/src/Controller/Courses/CoursesPutController.php) sería el punto de entrada para este caso de uso, recibiendo los campos necesarios para la creación de un nuevo curso. El será el responsable de llamar al caso de uso y comunicarse de vuelta con el cliente (Usando el standard HTTP). Tal como vemos en el constructor, recibiremos el caso de uso por Inyección de Dependencia

Implementando el Test ✅
-----------------------

Ya que el test que vamos a crear es sobre un caso de uso (Application Service), lo guardaremos dentro de la carpeta Application (Recordad las diferentes capas en Arquitectura Hexagonal) del módulo de Cursos que mimifica a la existente en _src/Mooc/_

Test `CourseCreatorTest`:

    final class CourseCreatorTest extends TestCase
    {
        /** @test */
        public function it_should_create_a_valid_course(): void
        {
            $repository = $this->createMock(CourseRepository::class);
            $creator    = new CourseCreator($repository);
            $id       = 'some-id';
            $name     = 'some-name';
            $duration = 'some-duration';
            $course = new Course($id, $name, $duration);
            $repository->method('save')->with($course);
            $creator->__invoke($id, $name, $duration);
        }
    }


Para validar el caso de uso tendremos que comprobar que realmente se llama al método `save` del repositorio que le vamos a inyectar, en ningún caso podremos basarnos en el retorno porque precisamente declararemos como _void_ el método para evidenciar que lo que produce es un side-effect.

Como comentábamos, no vamos a llegar a la implementación de la infraestructura, de hecho ni siquiera nos estamos preocupando sobre dónde se va a guardar, simplemente habrá un método `save` al que le pasaremos el agregado

Implementando el caso de uso
----------------------------

Clase `CourseCreator`:

    final class CourseCreator
    {
        private $repository;
    
    public function __construct(CourseRepository $repository)
        {
            $this->repository = $repository;
        }
    
    public function __invoke(string $id, string $name, string $duration)
        {
            $course = new Course($id, $name, $duration);
            $this->repository->save($course);
        }
    }


Tal como podíamos deducir desde el propio test, el caso de uso recibirá en el constructor el `CourseRepository` por inyección de dependencia. De momento el `__invoke` recibirá los tres parámetros en plano como strings (más adelante deberíamos encapsularlos en un DTO Request) El repositorio hablará siempre al mismo nivel de abstracción, por lo que le pasaremos (y nos devolverá si fuera el caso) el AggregateRoot con el que esté tratando. Es importante que no pervirtamos las responsabilidades de cada capa y por tanto establezcamos que sea el application service quien se ocupe de instanciar la clase

¡Con el caso de uso listo sólo queda lanzar los tests y ver si todo pasa!

¿Alguna Duda?
=============

Si tienes alguna duda sobre el contenido del video o quieres compartir alguna sugerencia no dudes en abrir una nueva discusión más abajo 👇👇👇

¡Nos vemos en el siguiente video! 👉 Implementación del repositorio en fichero y test de integración