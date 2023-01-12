😮 Casos de uso complejos que requieren información de varios contextos
========================================================================

Eliminar un curso
-----------------

*   Un usuario ha de poder borrar un curso
*   ✋ Con la restricción de que sólo se puede borrar un curso siempre y cuando haya más de 20

Clase `CourseRemover`:

    final class CourseRemover
    {
        private const MIN_COURSES_TO_ALLOW_REMOVE = 20;
    
        private $repository;
        private $finder;
        private $queryBus;
        private $eventBus;
    
        public function __construct(CourseRepository $repository, QueryBus $queryBus, EventBus $eventBus)
        {
            $this->repository = $repository;
            $this->finder     = new CourseFinder($repository);
            $this->queryBus   = $queryBus;
            $this->eventBus   = $eventBus;
        }
    
        public function __invoke(CourseId $id)
        {
            $response = $this->queryBus->ask(new FindeCoursesCounterQuery());
    
            if($response->total() > self::MIN_COURSES_TO_ALLOW_REMOVE)  {
                $course = $this->finder->find($id);
    
                $course->remove();
    
                $this->repository->remove($course);
                $this->eventBus->(publish(...$course->pullDomainEvents()));
            } else {
              throw new CourseCouldNotBeRemovedBecauseLimit(
                $id,
                $response->total(),
                self::MIN_COURSES_TO_ALLOW_REMOVE
              );
            }
        }
    }


Este Application Service habría sido llamado desde un Controller por medio de un comando en el que enviaría el _id_ del curso que se pretende borrar. Si vemos el constructor de la clase, estamos por un lado inyectando el repositorio y por otro instanciando un nuevo `CourseFinder`, esto es así puesto que la comunicación que va a producirse se engloba **dentro del mismo módulo y afecta al mismo agregado**, por lo que **no sería necesario lanzar ninguna Query**

Para recuperar el contador de total de cursos si que tenemos que comunicarnos con otro módulo y ya no es el mismo agregado, por lo que aquí si que lanzamos una Query. En caso de que el total sea menor que el mínimo que hemos establecido lanzaremos una excepción indicando que no se ha podido borrar el curso por esta razón.

En caso de cumplirse la condición del límite de cursos **recuperamos el agregado** Curso con su `finder` y, en primer lugar llamamos al _remove()_ del agregado que simplemente creará el Evento de Dominio ‘Curso Eliminado’. Una vez que lo hayamos eliminado del repositorio (pasándole el agregado, no el id que recibimos en la petición) publicaremos el evento para dejar constancia siempre de los cambios producidos

**CodelyTv Tip** ☝️: Si permitiésemos que el repositorio aceptase el identificador de curso estaríamos aceptando que un servicio de aplicación pudiera llamar al método de borrar curso sin haber instanciado el curso y sin poder publicar el evento de ‘Curso Eliminado’

Recuperar el identificador de recurso recién creado
---------------------------------------------------

Otro caso que podemos encontrarnos es que tratemos con un clientes legacy que tiren de una API anterior y necesiten que en la confirmación de creación del recurso les devuelvan el identificador de dicho recurso (Por lo que no podríamos llevarlo a un comando de CQRS de ‘crear curso’)

Una opción para este caso en el que no nos está llegando el Uuid desde fuera es, para al menos evitar que se genere por BD, generarlo en el propio Controller y añadirlo al propio recurso que nos habían enviado en la Request, de este modo facilitaremos que posteriormente podamos ir migrando este código más fácilmente

¿Alguna Duda?
=============

Si tienes alguna duda sobre este video o quieres compartir alguna sugerencia no dudes en abrir una nueva discusión más abajo 👇👇👇

¡Nos vemos en la siguiente Lección: Preguntas del Mundo Real™️!