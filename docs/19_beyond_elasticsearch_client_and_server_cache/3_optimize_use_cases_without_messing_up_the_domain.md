🦷 Cache de servidor: Optimizar casos de uso sin ensuciar el dominio
====================================================================

Tal como comentamos en el video anterior, optamos por la alternativa de situar la caché en la parte de infraestructura así que veremos cómo podemos implementarla ‘en memoria’ para la búsqueda de Cursos en el backoffice

*   En el caso de PHP, la Caché en memoria no es algo realmente cierto 🙊, ya que sólo estaría ‘viva’ durante la petición. En este caso delegaríamos la caché a un Redis

Clase `InMemoryCacheBackofficeCourseRepository` (Resultado final):

    final class InMemoryCacheBackofficeCourseRepository implements BackofficeCourseRepository
    {
        private static $allCoursesCache = [];
        private static $matchingCache = [];
        private $repository;
    
        public function __construct(BackofficeCourseRepository $repository)
        {
            $this->repository = $repository;
        }
    
        public function save(BackofficeCourse $course): void
        {
            $this->repository->save($course);
        }
    
        public function searchAll(): array
        {
            return empty(self::$allCoursesCache) ? $this->searchAllAndFillCache() : self::$allCoursesCache;
        }
    
        public function matching(Criteria $criteria): array
        {
            return get($criteria->serialize(), self::$matchingCache) ?: $this->searchMatchingAndFillCache($criteria);
        }
    
        private function searchAllAndFillCache(): array
        {
            return self::$allCoursesCache = $this->repository->searchAll();
        }
    
        private function searchMatchingAndFillCache(Criteria $criteria): array
        {
            return self::$matchingCache[$criteria->serialize()] = $this->repository->matching($criteria);
        }
    }


Con este wrapper lo que haremos será envolver cualquiera de las implementaciones de repositorio (MySQL, Elasticsearch…) que estemos utilizando (y que inyectaremos en el constructor), por lo que tendremos que implementar los métodos definidos en el contrato de la interface

A la hora de buscar todos los cursos, simplemente comprobaremos si tenemos los cursos ya en caché y, de lo contrario, los recuperaremos de BD tanto para guardarlos en caché como para devolverlos en la respuesta de la petición

En el caso del Matching trabajaremos con una caché a modo de clave-valor donde la clave sea el **Criteria serializado** y el valor el resultado de esa búsqueda filtrada. Esto nos llevará a serializar nuestro objeto Criteria y las estructuras subyacentes (Empujamos la lógica de cómo debe serializarse dentro del propio Agregado/Value Object)

Finalmente, como vimos en videos anteriores, será desde la definición de servicios donde le indicaremos que sea este wrapper el que nos inyecte cuando alguien esté solicitando la interface del repositorio

¿Alguna Duda?
=============

Si tienes alguna duda sobre este video o quieres compartir alguna sugerencia no dudes en abrir una nueva discusión más abajo 👇👇👇

¡Nos vemos en la siguiente lección: 👋 Autorización!