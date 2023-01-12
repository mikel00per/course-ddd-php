Patrón ObjectMother para nuestros tests
=======================================

Vamos a ver cómo implementar el patrón **ObjectMother** en nuestros tests unitarios. Ya vimos en qué consistía este patrón en el curso de [Testing: Introducción y buenas prácticas](https://pro.codely.tv/library/testing-introduccion-y-buenas-practicas/90916/about/), por lo que iremos al grano y nos centraremos en cómo añadirlos a nuestro proyecto PHP

*   Podéis encontrar el código relativo a este video en el tag [0.10.0 del repo](https://github.com/CodelyTV/php-ddd-skeleton/tree/0.10.0)

Test `FileCourseRepositoryTest`:

    final class FileCourseRepositoryTest extends TestCase
    {
        /** @test */
        public function it_should_save_a_course(): void
        {
            $repository = new FileCourseRepository();
            $course     = CourseMother::random();
            $repository->save($course);
        }
        
        // ....
    }



La principal diferencia con el diseño previo de nuestro código es que nuestros tests van a ser mucho más concisos y con mucho menos ruido. A simple vista vemos cómo estamos pasando a llamar a una fución _random()_ del propio `CourseMother` que nos devolverá una instancia totalmente random de Curso

Clase `CourseMother`:

    final class CourseMother
    {
        public static function create(CourseId $id, CourseName $name, CourseDuration $duration): Course
        {
            return new Course($id, $name, $duration);
        }
        public static function fromRequest(CreateCourseRequest $request): Course
        {
            return self::create(
                CourseIdMother::create($request->id()),
                CourseNameMother::create($request->name()),
                CourseDurationMother::create($request->duration())
            );
        }
        public static function random(): Course
        {
            return self::create(CourseIdMother::random(), CourseNameMother::random(), CourseDurationMother::random());
        }
    }


Un aspecto muy interesante en la implementación del ObjectMother es que la asignación de los valores de los **Value Objects** la delegará de nuevo en ObjectsMother para éstos (CourseIdMother, CourseNameMother, CourseDurationMother)

Todos estos ObjectsMother mimifican la estructura de producción por lo que dentro de la carpeta test se encontrarán en la carpeta de Domain

Clase `UuidMother`:

    final class UuidMother
    {
        public static function random(): string
        {
            return MotherCreator::random()->unique()->uuid;
        }
    }


Si observamos `CourseIdMother` encontraremos que a su vez esta llamando al ObjectMother de Uuid. En este caso estamos haciendo uso de nuevo de una dependencia externa, concretamente de **Faker**, una librería de PHP que nos permite generar valores aleatorios 🤹‍ (Tal como vimos en el Value Object de Uuid, nos valdremos de los ObjectMother para encapsular esta ‘contaminación’ de una librería externa)

Esta aleatorización en los tests nos forzará a que o bien controlemos todas las posibles acepciones en el código, o bien a limitar el tipo de valores aleatorios generados en el object mother

Tips para agilizar y customizar 🏃‍
-----------------------------------

Para facilitar el desarrollo de este tipo de elementos en nuestro código podemos utilizar templates generados con el propio IDE. Por ejemplo, nosotros hemos creado una _live template_ con PHP Velocity en IntelliJ IDEA que nos genere los métodos _create()_ y _random()_ instanciando la clase a partir del nombre del ObjectMother 🧙‍

Otro tip que nos puede ayudar en el desarrollo de pruebas es el hecho de declarar métodos en el ObjectMother que nos devuelvan valores específicos para cuando necesitemos realizar pruebas bajo condiciones concretas. Cabe decir que si creamos un método de este tipo por ejemplo en el `CourseNameMother`, deberíamos crear una función en el `CourseMother` que la llamase (Ley de Demeter)

¿Alguna Duda?
=============

Si tienes alguna duda sobre este video o quieres compartir alguna sugerencia no dudes en abrir una nueva discusión más abajo 👇👇👇

¡Nos vemos en el siguiente video Tests más semánticos 📖!