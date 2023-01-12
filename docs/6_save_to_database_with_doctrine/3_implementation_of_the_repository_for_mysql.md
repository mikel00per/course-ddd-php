# Implementación del repositorio para MySQL y test de integración

Una vez que ya hemos visto cómo crear el ‘esqueleto’ necesario para integrar Doctrine en nuestra aplicación, vamos a bajar a la implementación del repositorio para MySQL y montar los tests de integración correspondientes

*   Podéis encontrar el código relativo a este video en el tag [0.13.0 del repo](https://github.com/CodelyTV/php-ddd-skeleton/tree/0.13.0)

Testeando CourseRepository ✅
----------------------------

En este caso, puesto que lo que queremos testear es la implementación concreta del repositorio, no tendremos que mockear o falsear nada dentro de nuestra suite de tests. Lo que haremos será utilizar los métodos de dicha implementación y comprobar si realmente está haciendo lo que debería

Clase `CoursesModuleInfrastructureTestCase`:

    abstract class CoursesModuleInfrastructureTestCase extends MoocContextInfrastructureTestCase
    {
        protected function repository(): CourseRepository
        {
            return $this->service(CourseRepository::class);
        }
    }


Lo que tenemos aquí es la clase intermedia que utilizaremos para los tests de Infraestructura del módulo de Cursos (Recordad que ya creamos una para los Tests unitarios de este mismo módulo). En este método _repository()_ estamos accediendo al Contenedor de Dependencias para recuperar la implementación de la interface `CourseRepository` que allí esté establecida. El hecho de que no podamos sobrescribir el constructor en el test es la razón por la cual nos vemos obligados a utilizar esta clase intermedia en la cual no nos queda otra opción que exponer el contendor de dependencias y permitir un acoplamiento poco fino

*   En caso de tener varias implementaciones, lo recomendado será especificarlo devolviendo un _new_ de la implementación que queramos utilizar, además de renombrar el el método _repository_ para dejar claro con cual estaremos tratando (por ejemplo `MySQLRepository`)

*   Dado que esta clase intermedia no contiene particularidades que se realicen antes o después de los tests, podríamos hacer un pull down y traernos el método _repository()_ directamente a los tests, evitando esa jerarquía de clases. Como en otros casos, esto puede ser una cuestión de gustos a evaluar dentro del equipo


Test `CourseRepositoryTest` (En el video como DoctrineCourseRepository):


    final class CourseRepositoryTest extends CoursesModuleInfrastructureTestCase
    {
        /** @test */
        public function it_should_save_a_course(): void
        {
            $course = CourseMother::random();
            $this->repository()->save($course);
        }
        /** @test */
        public function it_should_return_an_existing_course(): void
        {
            $course = CourseMother::random();
            $this->repository()->save($course);
            $this->assertEquals($course, $this->repository()->search($course->id()));
        }
        /** @test */
        public function it_should_not_return_a_non_existing_course(): void
        {
            $this->assertNull($this->repository()->search(CourseIdMother::random()));
        }
    }


*   Finalmente optamos por evitar el prefijo ‘Doctrine’, pues no nos aporta realmente información de la infraestructura en partircular que estamos testeando en esta suite de tests 🍭

Puesto que estamos tratando de hacer independientes los tests entre sí, aunque en el primer case estamos comprobando que se guarde un curso en BD, a la hora de comprobar si se recupera correctamente, volveremos a guardarlo primero para buscarlo después en base a su Id.

Otro aspecto importante es que si queremos evitar conflictos y colisiones al ejecutar estos tests, deberíamos preparar nuestro entorno antes (setUp) y después (tearDown) de cada TestCase, limpiando la base de datos, cerrando conexiones…

¿Alguna Duda?
=============

Si tienes alguna duda sobre este video o quieres compartir alguna sugerencia no dudes en abrir una nueva discusión más abajo 👇👇👇

¡Nos vemos en el siguiente video: Discriminator Map con Doctrine!