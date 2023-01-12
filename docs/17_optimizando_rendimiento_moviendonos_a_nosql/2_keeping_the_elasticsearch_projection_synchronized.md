♻️ Mantener sincronizada la proyección de Elasticsearch
=======================================================

Una vez configurado en nuestra aplicación, lo siguiente que queremos es mantener sincronizada la proyección de nuestra BD en Elasticsearch, lo cual supone llevar a cabo una primera importación de todo el histórico de datos antes de pasar a ‘alimentarla’ por medio del consumo de eventos

La forma más sencilla de importar todos los registros existentes en nuestra BD de MySQL es a través de un **comando de symfony**.

Clase `ImportCoursesToElasticsearchCommand`:

    final class ImportCoursesToElasticsearchCommand extends Command
    {
        private $mySqlRepository;
        private $elasticRepository;
    
        public function __construct(
            MySqlBackofficeCourseRepository $mySqlRepository,
            ElasticsearchBackofficeCourseRepository $elasticRepository
        ) {
            parent::__construct();
            $this->mySqlRepository   = $mySqlRepository;
            $this->elasticRepository = $elasticRepository;
        }
    
        public function execute(InputInterface $input, OutputInterface $output)
        {
            $courses = $this->mySqlRepository->searchAll();
            foreach ($courses as $course) {
                $this->elasticRepository->save($course);
            }
        }
    }


Este comando recibirá un repositorio de origen (MySQL) y otro de destino (Elasticsearch). En este caso es bueno ese acoplamiento al repositorio de MySQL puesto que justamente lo que queremos es que nos inyecte una instancia concreta de este repositorio y no dejarlo abierto a un parámetro que definamos en la configuración ⛓

Desde el método _execute()_ lo que haremos será simplemente recuperar todos los cursos existentes e iterarlos para persistir cada uno de ellos en elasticSearch. En el caso de que la entidad que recuperamos de MySQL no sea la misma que vamos a persistir en Elasticsearch, simplemente necesitaremos hacer el mapeo

*   Ojo 👀: Tal como hemos definido este comando todo irá fino mientras el número de registros no sea excesivamente alto, en el momento en que esto sea asi, deberíamos parametrizar el comando para establecer unos rangos de búsqueda mas acotados, evitando fallos por sobrecarga de memoria

Clase `ElasticsearchBackofficeCourseRepository`:

    final class ElasticsearchBackofficeCourseRepository extends ElasticsearchRepository implements BackofficeCourseRepository
    {
        protected function aggregateName(): string
        {
            return 'courses';
        }
    
        public function save(BackofficeCourse $course): void
        {
            $this->persist($course->id(), $course->toPrimitives());
        }
        
        //...
    }


Como vimos previamente, el método _persist()_ de nuestro repositorio de Elasticsearch esperaba como argumentos el `id` y un `plainBody` (array clave-valor de datos primitivos), para formatear este último argumento hacemos uso del _toPrimitives()_ del agregado. Este método puede ser tema de debate en cuanto a donde debería ir, en nuestro caso no nos resulta ‘extraño’ tenerlo dentro del agregado (además nos encontramos ante una lógica bastante sencilla en este caso en el que incluso los datos están guardándose como primitivos y no como Value Objects)

Una vez que tenemos todo el código listo y realizada esta primera importación, el siguiente paso será activar nuestro subscriber para que continue sincronizando la proyección en Elasticsearch cada vez que se publique un evento de ‘Curso creado’

Dependiendo del volumen de datos y de la complejidad asociada a la creación de los scripts, otra alternativa que podríamos plantearnos para la importación inicial es la de recuperar todos los eventos publicados desde el inicio de los tiempos y lanzárselos al subscriber directamente 🔊

¿Alguna Duda?
=============

Si tienes alguna duda sobre este video o quieres compartir alguna sugerencia no dudes en abrir una nueva discusión más abajo 👇👇👇

¡Nos vemos en el siguiente video: 🍃 Patrón Criteria/Specification con Elasticsearch!