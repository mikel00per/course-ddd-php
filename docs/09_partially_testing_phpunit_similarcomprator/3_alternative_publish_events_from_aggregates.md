🤔 Alternativa: Publicar eventos desde agregados
================================================

Hasta ahora hemos venido planteando que la publicación de eventos se realizara en el caso de uso, separándo esta lógica de la relativa a la creación de estos eventos. La alternativa que vemos a continuación lo que propone es llevar la publicación también dentro del agregado

Clase `Course`:

    final class Course
    {
        private $id;
        private $name;
        private $duration;
    
        public function __construct(CourseId $id, CourseName $name, CourseDuration $duration)
        {
            $this->id       = $id;
            $this->name     = $name;
            $this->duration = $duration;
        }
    
        public static function create(
            CourseId $id, 
            CourseName $name, 
            CourseDuration $duration,
            CourseRepository $repository,
            EventBus $bus
            ): self
        {
            $course = new self($id, $name, $duration);
    
            $repository->save($course);
            $bus->publish(new CourseCreatedDomainEvent($id->value(), $name->value(), $duration->value()));
    
            return $course;
        }
    
        // ...
    }


Lo que planteamos aquí es eliminar la persistencia del Curso y la publicación de sus eventos en el caso de uso. Por el contrario, estaríamos pasando en el constructor del agregado los dos colaboradores que llevan a cabo estas acciones y haciendo que sea en el mismo momento en el que se realiza la mutación de Curso, se guarde en BD y se publique el evento.

Beneficios 🙋‍♀️
----------------

El hecho de publicar automáticamente **el evento de dominio** en el agregado supone que **ya no necesitamos almacenarlo**, por lo que tampoco nos hará falta extender de `AggregateRoot` en este caso

Si lanzamos en este momento los tests del caso de uso veremos que **todo sigue funcionando perfectamente**, puesto que lo que estamos comprobando aquí es que cuando se ejecuta el caso de uso se está llamando al _save()_ del repositorio y además se está publicando el evento de dominio, y eso sigue siendo asi! 🙌

Siguiendo con los tests, a la hora de comparar las estancias del Curso, aunque seguiremos necesitando utilizar el _similarTo()_ que vimos en el video anterior, si que podremos eliminar el `AggregateRootSimilarComparator` que nos veíamos obligados a añadir por el hecho de heredar de `AggregateRoot`

Inconvenientes 🙅‍♂
-------------------

Ahora bien, si nos situamos en el caso de estar llevando a cabo dos mutaciones sobre el agregado podemos encontrar algunos problemas a esta alternativa

En el caso de `CourseCounter` por ejemplo, lo que estaríamos haciendo es inicializar un nuevo contador primero (con su correspondiente guardado en BD y publicación de evento) para seguidamente incrementarlo. Pero si se diese un error al persistir en BD, el evento ya se habría publicado en el Bus, con lo cual los **subscribers adscritos a ese evento no podrían realizar un Rollback**

Podríamos controlar este tipo de situaciones, lo cual implicará que tengamos que añadir un Middleware que esté vigilando y sea quien finalmente publique los eventos si todo ha ido bien. El problema que esto supone es que éste **Middleware tendrá que conocer la implementación de EventBus** (y por tanto será dependiente de ella)

¿Alguna Duda?
=============

Nos encantaría conocer vuestra opinión de ambas alternativas en la publicación de eventos, así que no dudeis en abrir una nueva discusión más abajo para contarnos o para planteranos cualquier duda o sugerencia 👇👇👇

¡Nos vemos en la siguiente lección: 📥 Eventos de dominio asíncronos - MySQL!