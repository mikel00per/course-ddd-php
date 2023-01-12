🐣 Integración de Elasticsearch
===============================

En el momento en el que el volumen de registros sea bastante elevado vamos a encontrarnos con que realizar búsquedas en MySQL se vuelve algo muy lento y poco escalable, por lo que nos interesará utilizar un motor de búsqueda como Elasticsearch. Además, otro factor por el que nos interesará utilizarlo es que a nivel de UX sería mucho más óptimo tener un simple input donde introducir cualquier valor sin necesidad de añadir el campo y el tipo de operador tal y como veníamos haciendo en la lección anterior

Os recomendamos que, si no lo habéis realizado aún, os detengáis en este punto para hacer el [curso de Elasticsearch](https://pro.codely.tv/library/elkbeats-centraliza-la-gestion-de-logs-con-the-elastic-stack/about/) con el que podréis montaros todo el stack ELK y profundizar más en su funcionamiento. A partir de aquí veremos cómo vamos a integrarlo para realizar búsquedas indexadas aprovechando el Patrón Criteria que ya veníamos utilizando

Configurando Elasticsearch ⚙️
-----------------------------

    composer require elasticsearch/elasticsearch


En primer lugar, para poder disponer del cliente de Elasticsearch añadiremos la dependencia para utilizar esta libería en composer

configuración `course.json`:

    {
      "mappings": {
        "courses": {
          "properties": {
            "id": {
              "type": "keyword",
              "index": true
            },
            "name": {
              "type": "text",
              "index": true
            },
            "duration": {
              "type": "text",
              "index": true
            }
          }
        }
      }
    }


Por otra parte, evitaremos cualquier tipo de conflicto con los tipos de datos definiendo nosotros mismos el esquema, indicando el tipo para cada propiedad. Aunque por defecto el valor de ‘index’ es true, se lo indicamos igualmente para hacerlo más explícito, definiéndolo a false en aquellos casos en los que queramos recuperar un campo pero no realizar una búsqueda a través del mismo

Integramos Elasticsearch 🍭
---------------------------

Una vez configurado, lo que queremos hacer es crear un índice por cada recurso que necesitemos indexar/buscar

Hemos creado [este wrapper](https://github.com/CodelyTV/php-ddd-skeleton/blob/master/src/Shared/Infrastructure/Elasticsearch/ElasticsearchClient.php) en la infraestructura compartida por los distintos Bounded Context que recoje tanto el cliente de Elasticsearch como el prefijo del contexto y será a través de este wrapper con el que interactuaremos. Puesto que para inicializar el cliente necesitamos algunos atributos de configuración y no podemos hacer un autowiring sin más, hemos creado una Factoría que se encargue de esta tarea

Clase `ElasticsearchClientFactory`:

    final class ElasticsearchClientFactory
    {
        public function __invoke(
            string $host,
            string $indexPrefix,
            string $schemasFolder,
            string $environment
        ): ElasticsearchClient {
            $client = ClientBuilder::create()->setHosts([$host])->build();
            $this->generateIndexIfNotExists($client, $indexPrefix, $schemasFolder, $environment);
            return new ElasticsearchClient($client, $indexPrefix);
        }
    
        private function generateIndexIfNotExists(
            Client $client,
            string $indexPrefix,
            string $schemasFolder,
            $environment
        ): void {
            $indexes = Utils::filesIn($schemasFolder, '.json');
            foreach ($indexes as $index) {
                $indexName = str_replace('.json', '', sprintf('%s_%s', $indexPrefix, $index));
                if ('prod' !== $environment && !$this->indexExists($client, $indexName)) {
                    $indexBody = Utils::jsonDecode(file_get_contents("$schemasFolder/$index"));
                    $client->indices()->create(['index' => $indexName, 'body' => $indexBody]);
                }
            }
        }
    
        private function indexExists(Client $client, string $indexName): bool
        {
            try {
                $client->indices()->getSettings(['index' => $indexName]);
                return true;
            } catch (Missing404Exception $unused) {
                return false;
            }
        }
    }


El método _invoke()_ recibe los parámetros desde el fichero de configuración ‘services.yaml’ de backoffice/frontend, donde le especificamos qué variables de entorno debemos pasarle a la factoría cuando se quiera instanciar la clase ElasticsearchClient (recordemos que está dentro de las buenas prácticas de PHP y Symfony seguir este procedimiento para el paso de argumentos)

Dentro del invoke le indicamos que nos genere los índices (definir la estructura) en caso de no existir ya y estar en un entorno distinto a producción (para así evitar tener que generarlos a mano para los tests)

Finalmente tendremos el repositorio `ElasticsearchRepository` (Del mismo modo que veíamos con Doctrine) con sus propios métodos donde usaremos el cliente de Elasticsearch para insertar/buscar, además este repositorio nos obliga a definir el `aggregateName`, que nos facilitará precisamente la gestión de los índices

¿Alguna Duda?
=============

Si tienes alguna duda sobre este video o quieres compartir alguna sugerencia no dudes en abrir una nueva discusión más abajo 👇👇👇

¡Nos vemos en el siguiente video: ♻️ Mantener sincronizada la proyección de Elasticsearch!