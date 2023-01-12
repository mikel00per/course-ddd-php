Utilizando objetos Request y Response para comunicarnos con el Application Service
==================================================================================

Tal como ya veíamos en los cursos de [SOLID](https://pro.codely.tv/library/principios-solid-aplicados/77070/about/), [Arquitectura Hexagonal](https://pro.codely.tv/library/arquitectura-hexagonal/66748/about/) y [DDD Aplicado](https://pro.codely.tv/library/domain-driven-design-ddd/87157/about/), el Controlador suele comunicarse con el Servicio de Aplicación a través de un DTO (Data Transfer Object), encapsulando los escalares recibidos en la petición, lo cual además nos facilitará una posible migración a CQRS más adelante

Crearemos este DTO Request en la capa de aplicación, ya que será aquí donde permitiremos exponer una api sobre las cosas que podemos hacer dentro del subdominio

*   Podéis encontrar el código relativo a este video en el tag [0.9.0 del repo](https://github.com/CodelyTV/php-ddd-skeleton/tree/0.9.0)

Clase `CreateCourseRequest`:

    final class CreateCourseRequest
    {
        private $id;
        private $name;
        private $duration;
    
        public function __construct(string $id, string $name, string $duration)
        {
            $this->id       = $id;
            $this->name     = $name;
            $this->duration = $duration;
        }
    
        public function id(): string
        {
            return $this->id;
        }
        
        public function name(): string
        {
            return $this->name;
        }
        
        public function duration(): string
        {
            return $this->duration;
        }
    }


Definiremos los atributos de clase de forma privada y los inicializaremos en el constructor de forma que nadie pueda modificar su estado y simplemente generaremos los getter de los atributos que por convención, en nuestro caso, llamaremos con el mismo nombre que el atributo que devuelven

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
            $this->creator->__invoke(
                new CreateCourseRequest(
                    $id,
                    $request->request->get('name'),
                    $request->request->get('duration')
                )
            );
            return new Response('', Response::HTTP_CREATED);
        }
    }


Ahora nuestro Controlador en lugar de los escalares podrá enviar como argumento una `CreateCourseRequest`. Será necesario actualizar también nuestro test unitario, en el caso del test de aceptación no será necesario ya que lo único que éste conoce es el punto de entrada

¿Alguna Duda?
=============

Si tienes alguna duda sobre el contenido del video o quieres compartir alguna sugerencia no dudes en abrir una nueva discusión más abajo 👇👇👇

¡Nos vemos en el siguiente video! 👉 Refactoring a UUIDs como identificadores