🐋 Filtrado de elementos: Patrón Criteria en MySQL
==================================================

Hemos creado un objeto Criteria que forma parte de nuestro Dominio y ahora veremos cómo transformarlo en una Query de MySQL a través de Doctrine, lo que podemos entender como el concepto de Puertos-Adaptadores de la Arquitectura Hexagonal

Clase `SearchBackofficeCoursesByCriteriaQueryHandler`:

    final class SearchBackofficeCoursesByCriteriaQueryHandler implements QueryHandler
    {
        private $searcher;
        
        public function __construct(BackofficeCoursesByCriteriaSearcher $searcher)
        {
            $this->searcher = $searcher;
        }
    
        public function __invoke(SearchBackofficeCoursesByCriteriaQuery $query): BackofficeCoursesResponse
        {
            $filters = Filters::fromValues($query->filters());
            $order   = Order::fromValues($query->orderBy(), $query->order());
    
            return $this->searcher->search($filters, $order, $query->limit(), $query->offset());
        }
    }


El QueryHandler asociado a esta Query realiza dos acciones importantes una vez que se invoca: En primer lugar mapeará la colección de Filtros y el Orden a partir de la Query, y en sengudo lugar llamará al método _search()_ del caso de uso

Para recuperar la colección de filtros utilizamos el método estático _fromValues()_ perteneciente a la propia clase `Filters`, que hará un map con el array de filtros de la query y para cada uno de ellos llamará al _fromValues()_ de `Filter` creando un nuevo filtro

En el caso del orden igualmente haremos uso de un método estático de `Order` para componerlo en el caso de que nos lleguen estos valores en la query

Clase `BackofficeCoursesByCriteriaSearcher`:

    final class BackofficeCoursesByCriteriaSearcher
    {
        private $repository;
        public function __construct(BackofficeCourseRepository $repository)
        {
            $this->repository = $repository;
        }
        public function search(Filters $filters, Order $order, ?int $limit, ?int $offset): BackofficeCoursesResponse
        {
            $criteria = new Criteria($filters, $order, $offset, $limit);
            return new BackofficeCoursesResponse(...map($this->toResponse(), $this->repository->matching($criteria)));
        }
        private function toResponse(): callable
        {
            return static function (BackofficeCourse $course) {
                return new BackofficeCourseResponse($course->id(), $course->name(), $course->duration());
            };
        }
    }


Es en el método dentro del caso de uso donde instanciamos nuestro objeto `Criteria` para poder enviárselo al repositorio, transformando finalmente la respuesta de éste en un listado de `BackofficeCourseResponse` que es precisamente lo que el controlador espera recibir (nuestro `BackofficeCoursesResponse` implementa la interface `Response`)

Internamente el método _matching()_ del repositorio utiliza el Criteria de Doctrine, por lo que primeramente hacemos uso de nuestro `DoctrineCriteriaConverter` que mapea [de este modo](https://github.com/CodelyTV/php-ddd-skeleton/blob/master/src/Shared/Infrastructure/Persistence/Doctrine/DoctrineCriteriaConverter.php#L28-L36) nuestro `Criteria` al formato de Doctrine. Con esta transformación hecha, se la pasamos al _matching()_ propio de Doctrine y devolvemos el resultado en un array (ya que por defecto lo que devolvería sería un iterador propio)

*   Ojo 👀: Estamos haciendo uso de métodos estáticos, que de base podemos entender como algo nocivo en nuestro código, pero al ser utilizados sólo en la implementación de Infraestructura entendemos que es un acoplamiento que va a estar siempre encapsulado aquí

Test `MySqlBackofficeCourseRepositoryTest`:

    final class MySqlBackofficeCourseRepositoryTest extends BackofficeCoursesModuleInfrastructureTestCase
    {
        // ...
        
        /** @test */
        public function it_should_search_all_existing_courses_with_an_empty_criteria(): void
        {
            $existingCourse        = BackofficeCourseMother::random();
            $anotherExistingCourse = BackofficeCourseMother::random();
            $existingCourses       = [$existingCourse, $anotherExistingCourse];
    
            $this->repository()->save($existingCourse);
            $this->repository()->save($anotherExistingCourse);
    
            $this->clearUnitOfWork();
    
            $this->assertSimilar($existingCourses, $this->repository()->matching(CriteriaMother::empty()));
        }
    
        /** @test */
        public function it_should_filter_by_criteria(): void
        {
            $dddInPhpCourse  = BackofficeCourseMother::withName('DDD en PHP');
            $dddInJavaCourse = BackofficeCourseMother::withName('DDD en Java');
            $intellijCourse  = BackofficeCourseMother::withName('Exprimiendo Intellij');
            
            $dddCourses      = [$dddInPhpCourse, $dddInJavaCourse];
    
            $nameContainsDddCriteria = BackofficeCourseCriteriaMother::nameContains('DDD');
    
            $this->repository()->save($dddInJavaCourse);
            $this->repository()->save($dddInPhpCourse);
            $this->repository()->save($intellijCourse);
    
            $this->clearUnitOfWork();
    
            $this->assertSimilar($dddCourses, $this->repository()->matching($nameContainsDddCriteria));
        }
    }


A nivel de Tests comprobamos tanto una llamada sin ningún filtro (en tal caso nos debe devolver todos los cursos existentes) como la llamada pasándole filtros concretos. En este caso hemos definido Objects Mother más específicos que nos permitan generar filtros acordes a los campos que contiene el elemento que buscamos (No olvidemos añadir al final la llamada al _clearUnitOfWork()_ !!)

Puesto que no especificamos ningún orden, éste puede variar en las distintas ejecuciones de los tests, por lo que usamos la aserción _assertSimilar_ que compara sin tener en cuenta el orden de los elementos y no _assertEquals_

¿Alguna Duda?
=============

Si tienes alguna duda sobre este video o quieres compartir alguna sugerencia no dudes en abrir una nueva discusión más abajo 👇👇👇

¡Nos vemos en la siguiente lección: 💡 Optimizando rendimiento - Moviéndonos a NoSQL: Elasticsearch al rescate!