Repositorios con Eloquent: Escondiendo en la infraestructura la contaminación
=============================================================================

Una vez configurado en nuestra aplicación veremos cómo trabajar con Eloquent, el ORM que utiliza Laravel por defecto, y cómo esconder en la capa de infraestructura toda la contaminación que este implica

El patrón **Active Record** que implementa Laravel con Eloquent nos permite desarrollar a un ritmo bastante rápido y puede resultar interesante en aplicaciones que no vayan a escalar demasiado. Sin embargo, como veíamos en el video anterior, esta rapidez se consigue a costa de ‘ensuciar’ nuestro dominio

*   Podéis encontrar el código relativo a este video en el tag [0.15.0 del repo](https://github.com/CodelyTV/php-ddd-skeleton/tree/0.15.0)

Clase `CourseCreator`:

    final class CourseCreator
    {
        private $repository;
        
        public function __construct(CourseRepository $repository)
        {
            $this->repository = $repository;
        }
    
        public function __invoke(CreateCourseRequest $request)
        {
            $id       = new CourseId($request->id());
            $name     = new CourseName($request->name());
            $duration = new CourseDuration($request->duration());
    
            $course = new Course($id, $name, $duration);
            
            $this->repository->save($course);
        }
    }


Si analizamos el caso de uso, veremos que permanece exáctamente igual que si trabajásemos con Doctrine: La capa de Aplicación no se está viendo contaminada por las dependencias externas y lo que estamos pasando al repositorio es un Curso de nuestro Dominio (no uno que herede de Model)

Tampoco permitiremos que el contrato establecido por `CourseRepository` se vea afectado, por lo que estableceremos la barrera a nivel de la implementación del repositorio. De este modo, además de **evitar** ese **acoplamiento** con Eloquent, estamos ganando en **estandarización de nuestra aplicación** al permitir que se mantenga la misma estructura de carpetas definidas siguiendo Arquitectura Hexagonal

clase `EloquentCourseRepository`:

    final class EloquentCourseRepository implements CourseRepository
    {
        public function save(Course $course): void
        {
            $model           = new CourseEloquentModel();
            $model->id       = $course->id()->value();
            $model->name     = $course->name()->value();
            $model->duration = $course->duration()->value();
            $model->save();
        }
        public function search(CourseId $id): ?Course
        {
            $model = CourseEloquentModel::find($id->value());
    
            if (null === $model) {
                return null;
            }
            return new Course(new CourseId($model->id), new CourseName($model->name), new CourseDuration($model->duration));
        }
    }


Como Eloquent necesita que una clase extienda de Model para poder realizar su mapping automático, la solución que hemos planteado es crear en la carpeta de Infraestructura una clase `CourseEloquentModel` que será la que extienda de Model y le seteamos los atributos de nuestro agregado Curso

En el caso del método _search_ llevamos a cabo el proceso contrario, si recuperamos un CourseEloquentModel de BD lo que haremos será devolver una nueva instancia de `Course` a la que le setearemos los atributos del model

A nivel de nuestros [tests unitarios](https://github.com/CodelyTV/php-ddd-skeleton/blob/0.14.0/tests/src/Mooc/Courses/Infrastructure/Persistence/CourseRepositoryTest.php) tampoco nos estaremos viendo contaminados por la implementación de Eloquent en el repositorio ya que tal como vimos anteriormente no nos estamos acoplando en ellos a ninguna implementación concreta, por lo que podremos mockearla sin mayor inconveniente

¿Alguna Duda?
=============

¡Hasta aquí la integración de nuestra aplicación con Eloquent! En nuestra opinión podemos concluir que para desarrollos rápidos de aplicaciones la opción será Eloquent, pero deberíamos considerar utilizar Doctrine si lo que buscamos es testabilidad, mantenibilidad, tolerancia al cambio ya que de lo contrario nos encontraremos con un esfuerzo extra llevando a cabo el enmascaramiento que hemos visto en este video

Si tienes alguna duda sobre este video o quieres compartir alguna sugerencia no dudes en abrir una nueva discusión más abajo 👇👇👇

¡Nos vemos en la siguiente lección: 📤 Eventos de dominio síncronos con Symfony Messenger: Incrementar el total de cursos!