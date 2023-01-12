Tests más semánticos
====================

Aunque ya hemos dejado bastante finos nuestros tests, puliéndolos en términos de instanciaciones y agregados, aún podemos mejorar un poco más nuestros tests y dejarlos más limpios

*   Podéis encontrar el código relativo a este video en el tag [0.11.0 del repo](https://github.com/CodelyTV/php-ddd-skeleton/tree/0.11.0)

Test `CourseCreatorTest` previo:

    final class CourseCreatorTest extends TestCase
    {
        /** @test */
        public function it_should_create_a_valid_course(): void
        {
            $repository = $this->createMock(CourseRepository::class);
            $creator    = new CourseCreator($repository);
       
            $request = CreateCourseRequestMother::random();
            
            $course  = CourseMother::fromRequest($request);
            
            $repository->method('save')->with($course);
            
            $creator->__invoke($request);
        }
    }


A la hora de probar este caso de uso de ‘Crear un nuevo Curso’, lo que realmente nos interesa es el _expect_, saber que se realiza correctamente la llamada al método _save()_ pasándole un curso, las instanciaciones previas no nos aportan nada que queramos ver al acceder al Test. Lo que buscaremos entonces será reducir al mínimo posible ese ‘ruído’ dentro del Test 🤫

Con PHPUnit nos encontramos que no podemos hacer uso del constructor por lo que, en su lugar, utilizaremos el método _setUp()_ que se ejecuta automáticamente antes de cada Test

Test `CourseCreatorTest` con setUp:

    final class CourseCreatorTest extends CoursesModuleUnitTestCase
    {
        private $repository
        private $creator;
        protected function setUp(): void
        {
            parent::setUp();
    
            $this->creator = new CourseCreator($this->repository());
        }
        /** @test */
        public function it_should_create_a_valid_course(): void
        {
            $request = CreateCourseRequestMother::random();
    
            $course = CourseMother::fromRequest($request);
            
            $this->repository()->method('save')->with($course)
            
            $this->creator->__invoke($request);
        }
    
        /** @return CourseRepository|MockObjeck */
        private function repository(): MockObject
        {
            return $this->repository = $this->repository ? : $this->createMock(CourseRepository::class);
        }
    }


Lo que haremos será **llevarnos** a este método _setUp()_ **todo el ruido de instanciación** del caso de uso y del repositorio. Además, para evitar estar creando constantemente instanciaciones del repositorio, utilizaremos una función auxiliar _repository()_ que devolverá una nueva instancia sólo en el caso de que no exista ya una. Es interesante que como `CourseCreator` espera una instancia de `CourseRepository`, tendremos que indicar que el tipo de retorno de _repository()_ puede ser tanto éste como un `MockObject`, pues lo que realmente devuelve el método es un mock del repositorio

Por otro lado, también extraeremos a un método más semántico como _shouldSave()_ la llamada que realizamos a guardar el curso en el repositorio. Este planteamiento va muy ligado al planteamiento de DDD de tratar de **agregados lo más pequeños posible** para así poder seguir empujando la lógica de dominio hasta lo más profundo, esto nos llevará por lo general a tener un único repositorio por módulo

Test `CourseCreatorTest` heredando de `CoursesModuleUnitTestCase`:

    final class CourseCreatorTest extends CoursesModuleUnitTestCase
    {
        private $creator;
        protected function setUp(): void
        {
            parent::setUp();
    
            $this->creator = new CourseCreator($this->repository());
        }
        /** @test */
        public function it_should_create_a_valid_course(): void
        {
            $request = CreateCourseRequestMother::random();
    
            $course = CourseMother::fromRequest($request);
            
            $this->shouldSave($course);
            
            $this->creator->__invoke($request);
        }
    }


Aunque habíamos limpiado bastante nuestro test, aún teníamos código que además de generar ruído en el Test, será fácilmente reutilizable por los demás TestCase que creemos dentro de este módulo. Para mejorar este aspecto, lo que haremos será crear una clase abstracta `CoursesModuleUnitTestCase`de la que herede el Test (Recordemos que con PHPUnit no podemos inyectar colaboradores en el constructor) 👨‍👦

Clase `CoursesModuleUnitTestCase`:

    abstract class CoursesModuleUnitTestCase extends TestCase
    {
        private $repository;
    
        protected function shouldSave(Course $course): void
        {
            $this->repository()->method('save')->with($course);
        }
    
        /** @return CourseRepository|MockObject */
        protected function repository(): MockObject
        {
            return $this->repository = $this->repository ?: $this->createMock(CourseRepository::class);
        }
    }


Tendremos una de estas clases por cada módulo dentro del Bounded Context dentro de la carpeta de Infraestructura con los dobles de Tests de cada módulo

Discrepancias en el diseño 🔪
-----------------------------

Frente a las ventajas que hemos visto en esta manera de plantear los tests, algo que puede generar cierta discrepancia es que, precisamente, **nos dificulta** en cierto modo **la trazabilidad** de todo lo que está pasando y el ver qué tipo de instanciaciones y métodos están interactuando. En este sentido podríamos optar por hacer explícita la instanciación siguiendo patrones como Given-When-Then o Arrange-Act-Assert como veíamos en el curso de [Testing](https://pro.codely.tv/library/testing-introduccion-y-buenas-practicas/90916/about/)

Por supuesto al final dependerá de con qué planteamiento nos encontremos más cómodos o cual es la opción que menos chirría dentro del equipo de desarrollo

¿Alguna Duda?
=============

Si tienes alguna duda sobre este video o quieres compartir alguna sugerencia no dudes en abrir una nueva discusión más abajo 👇👇👇

¡Nos vemos en la siguiente lección: 🕋 Guardar en base de datos con Doctrine!