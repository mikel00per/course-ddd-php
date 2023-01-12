🍃 Patrón Criteria/Specification con Elasticsearch
==================================================

Ya sabemos cómo **integrar Elasticsearch** en nuestra aplicación y cómo **mantener una proyección** bien sincronizada de los datos por lo que únicamente nos queda por ver cómo **explotar los datos** utilizando el patrón Criteria que ya veníamos utilizando desde MySQL a través del propio objeto Criteria que Doctrine implementa

En el caso de Elasticsearch por tanto necesitaremos definir una clase Converter que mapee el Criteria de nuestro dominio a un cuerpo de petición que este motor de búsqueda entienda 🗣👂

Clase `ElasticsearchBackofficeCourseRepository`:

    final class ElasticsearchBackofficeCourseRepository extends ElasticsearchRepository implements BackofficeCourseRepository
    {
        
        // ...
        public function searchAll(): array
        {
            return map($this->toCourse(), $this->searchAllInElastic());
        }
    
        public function matching(Criteria $criteria): array
        {
            return map($this->toCourse(), $this->searchByCriteria($criteria));
        }
        
        private function toCourse()
        {
            return static function (array $primitives) {
                return BackofficeCourse::fromPrimitives($primitives);
            };
        }
    }


En este caso el método _matching()_ de esta implementación del repositorio llamará al _searchByCriteria()_ pasándole la instancia de Criteria y transformando cada elemento devuelto a agregados de tipo Curso a través del método estático _toPrimitives()_ de la clase `BackofficeCourse` que precisamente nos devuelve una instancia de esa clase a partir de los datos en primitivo (que es como los habíamos guardado en Elasticsearch utilizando el método _toPrimitives()_)

Por otra parte, desde el _searchbyCriteria()_ de `ElasticsearchRepository` será donde llamemos a [nuestro Converter](https://github.com/CodelyTV/php-ddd-skeleton/blob/master/src/Shared/Infrastructure/Persistence/Elasticsearch/ElasticsearchCriteriaConverter.php) para montar el body de la query y pasársela al método _searchRawElasticsearchQuery()_. Dentro de lo que nos devuelva Elasticsearch recogeremos lo que haya dentro de ‘hits->hits’ extrayendo un mapeo de los valores

Un detalle importante de esta llamada es que la hemos encapsulado en un bloque try-catch puesto que si no se encuentran resultados Elasticsearch lanza un error 404 que es traducido en una `Missing404Exception` por la librería de php para Elasticsearch de forma dinámica y nosotros no queremos que se lance una excepción (y menos aún una excepción de infraestructura 🙅) por no haber encontrado resultados en un ‘search’, sino que simplemente nos devuelva un array vacío

*   **CodelyTV Tip** ☝️: Es importante que nuestros **repositorios** sean consistentes y **hablen siempre al mismo nivel de abstracción**, es decir, van a recibir agregados que persistir y van a devolver agregados al realizar una búsqueda (Hay que tener especial atención en el caso de PHP al no poder tipar el tipo de array devuelto en una búsqueda, por la firma del método le dará igual si retornamos un array de Cursos o uno de datos primitivos)

¿Alguna Duda?
=============

Si tienes alguna duda sobre este video o quieres compartir alguna sugerencia no dudes en abrir una nueva discusión más abajo 👇👇👇

¡Nos vemos en la siguiente lección: 🛁 Limpiando deuda técnica: Separando Bounded Contexts!