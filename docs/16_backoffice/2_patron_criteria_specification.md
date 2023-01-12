🚁 Filtrado de elementos: Patrón Criteria/Specification
=======================================================

Ya estamos listando todos los cursos que hemos creado y el siguiente paso es poder filtrarlos, pero además queremos que el **número de filtros** aplicados sea algo **dinámico** y que el hecho de tener una combinatoria elevada de éstos no se traduzca en un repositorio con un listado inmenso de métodos que cubran estas posibilidades

Para implementar este tipo de consulta haremos uso del querido Patrón Criteria o Specificattion Pattern

Clase `BackofficeCourseRepository`:

    interface BackofficeCourseRepository
    {
        public function save(BackofficeCourse $course): void;
    
        public function searchAll(): array;
    
        public function matching(Criteria $criteria): array;
    }


Este patrón nos permite mantener el repositorio mucho más limpio, definiendo un único método _matching()_ que espera recibir un objeto `Criteria` que contendrá una combinación (array) de **filtros**, un **orden** y opcionalmente **offset** y **limit** por si quisiéramos limitar el número de registros a recuperar

Por su parte, cada `Filter` estará compuesto por un **campo** (Nombre, Duración, Id…), un **operador** (Contiene, Igual, Distinto…) y un **valor**

Este patrón nos va a facilitar enormemente la mantenibilidad de nuestros repositorios, por lo que es recomendable implementar este tipo de estrategias a no ser que tengamos claro que nuestras necesidades no van a ir más allá de dos o tres métodos dentro del repositorio

Clase `CoursesGetController`:

    final class CoursesGetController
    {
        private $queryBus;
    
        public function __construct(QueryBus $queryBus)
        {
            $this->queryBus = $queryBus;
        }
    
        public function __invoke(Request $request): JsonResponse
        {
            /** @var BackofficeCoursesResponse $response */
            $response = $this->queryBus->ask(
                new SearchBackofficeCoursesByCriteriaQuery(
                    $request->query->get('filters', []),
                    $request->query->get('order_by'),
                    $request->query->get('order'),
                    $request->query->get('limit'),
                    $request->query->get('offset')
                )
            );
    
            return new JsonResponse(
                map($this->toArray(), $response->courses()),
                200,
                ['Access-Control-Allow-Origin' => '*']
            );
        }
    
        // ...
    }


Para pasar el array de filtros en la petición al Controlador aprovechamos en este caso el estándar seguido por PHP para el envío de arrays vía url en peticiones GET. El string generado es parseado por Symfony de forma que podemos acceder a él desde la Query recogida en el objeto Request

¿Alguna Duda?
=============

Si tienes alguna duda sobre este video o quieres compartir alguna sugerencia no dudes en abrir una nueva discusión más abajo 👇👇👇

¡Nos vemos en el siguiente video: 🐋 Filtrado de elementos: Patrón Criteria en MySQL!