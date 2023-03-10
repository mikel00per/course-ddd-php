馃 Alternativa: Publicar eventos desde agregados
================================================

Hasta ahora hemos venido planteando que la publicaci贸n de eventos se realizara en el caso de uso, separ谩ndo esta l贸gica de la relativa a la creaci贸n de estos eventos. La alternativa que vemos a continuaci贸n lo que propone es llevar la publicaci贸n tambi茅n dentro del agregado

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


Lo que planteamos aqu铆 es eliminar la persistencia del Curso y la publicaci贸n de sus eventos en el caso de uso. Por el contrario, estar铆amos pasando en el constructor del agregado los dos colaboradores que llevan a cabo estas acciones y haciendo que sea en el mismo momento en el que se realiza la mutaci贸n de Curso, se guarde en BD y se publique el evento.

Beneficios 馃檵鈥嶁檧锔?
----------------

El hecho de publicar autom谩ticamente **el evento de dominio** en el agregado supone que **ya no necesitamos almacenarlo**, por lo que tampoco nos har谩 falta extender de `AggregateRoot` en este caso

Si lanzamos en este momento los tests del caso de uso veremos que **todo sigue funcionando perfectamente**, puesto que lo que estamos comprobando aqu铆 es que cuando se ejecuta el caso de uso se est谩 llamando al _save()_ del repositorio y adem谩s se est谩 publicando el evento de dominio, y eso sigue siendo asi! 馃檶

Siguiendo con los tests, a la hora de comparar las estancias del Curso, aunque seguiremos necesitando utilizar el _similarTo()_ que vimos en el video anterior, si que podremos eliminar el `AggregateRootSimilarComparator` que nos ve铆amos obligados a a帽adir por el hecho de heredar de `AggregateRoot`

Inconvenientes 馃檯鈥嶁檪
-------------------

Ahora bien, si nos situamos en el caso de estar llevando a cabo dos mutaciones sobre el agregado podemos encontrar algunos problemas a esta alternativa

En el caso de `CourseCounter` por ejemplo, lo que estar铆amos haciendo es inicializar un nuevo contador primero (con su correspondiente guardado en BD y publicaci贸n de evento) para seguidamente incrementarlo. Pero si se diese un error al persistir en BD, el evento ya se habr铆a publicado en el Bus, con lo cual los **subscribers adscritos a ese evento no podr铆an realizar un Rollback**

Podr铆amos controlar este tipo de situaciones, lo cual implicar谩 que tengamos que a帽adir un Middleware que est茅 vigilando y sea quien finalmente publique los eventos si todo ha ido bien. El problema que esto supone es que 茅ste **Middleware tendr谩 que conocer la implementaci贸n de EventBus** (y por tanto ser谩 dependiente de ella)

驴Alguna Duda?
=============

Nos encantar铆a conocer vuestra opini贸n de ambas alternativas en la publicaci贸n de eventos, as铆 que no dudeis en abrir una nueva discusi贸n m谩s abajo para contarnos o para planteranos cualquier duda o sugerencia 馃憞馃憞馃憞

隆Nos vemos en la siguiente lecci贸n: 馃摜 Eventos de dominio as铆ncronos - MySQL!