Integración con Doctrine
========================

Empezamos a subir el ritmo de este curso y lo hacemos de la mano de **Doctrine**. Veremos cómo se integra y las particularidades que tiene a la hora de trabajar con DDD.

*   Podéis encontrar el código relativo a este video en el tag [0.12.0 del repo](https://github.com/CodelyTV/php-ddd-skeleton/tree/0.12.0)

Doctrine es una librería en PHP que nos permite el **mapeo de entidades** para persistirlas en BD. Para este mapeo le indicaremos la información necesaria bien mediante anotaciones en las clases o bien mediante ficheros de configuración (xml, yaml). Puesto que no queremos ‘ensuciar’ las entidades añadiendo anotaciones y que se dejará de dar soporte a yaml, definiremos las relaciones con ficheros xml

    composer require doctrine/orm
    composer require doctrine/dbal


Para poder trabajar con Doctrine será necesario que requiramos las dependencias de _orm_ y _dbal_. La dependencia de _dbal_ nos va a permitir customizar cosas a más bajo nivel, mientras que _orm_ nos proveerá de toda la mágia como crear repositorios a partir de una clase

Clase `MySqlCourseRepository` (En el video como DoctrineCourseRepository):

    final class MySqlCourseRepository implements CourseRepository
    {
        private $entityManager;
        public function __construct(EntityManager $entityManager)
        
        {
            $this->entityManager = $entityManager;
        }
    
        public function save(Course $course): void
        {
            $this->entityManager->persist($course);
    
            $this->entityManager->flush($course);
        }
        
        public function search(CourseId $id): ?Course
        {
            return $this->entityManager->getRepository(Course::class)->find($id);
        }
    }


*   Así quedaría finalmente la clase `DoctrineCourseRepository` que puede verse en el video. Hemos consensuado que es preferible exponer de forma explícita en el nombre de la clase el detalle de infraestructura que hace particular a esta implementación (Al igual que nombraríamos la clase `InMemoryCourseRpository` cuando se trate de un repositorio en memoria)

El objetivo por el que usaremos Doctrine es tener un repositorio como este, que nos permita de forma sencilla guardar un Curso que recibamos por parámetro y devolver un Curso en base al identificador que nos pasen (Recordemos que al trabajar con la _Unit of Work_ necesitamos hacer el _persist()_ primero y después hacer uso del método _flush()_

Accederemos al EntityManager para recuperar un repositorio de Cursos, para lo cual le pasaremos la clase por parámetro con la sintaxis ‘::class’, con esto y los _xml_ de configuración Doctrine hará su magia e instanciará dicho repositorio.

Gracias a las convenciones que venimos siguiendo y que hemos ido consensuando será mucho más sencillo indicar a Doctrine donde podrá encontrar nuestros mappings: Dentro de la carpeta Infraestructura de cada modelo en cada uno de nuestros Bounded Contexts crearemos una carpeta ‘Persistencia’ y dentro de èsta otra carpeta ‘Doctrine’ en la que tendremos estos ficheros

La clase `EntityManager` que vemos en nuestro repositorio es la pieza principal de Doctrine y es quien nos provee de la Unit of Work en torno a nuestros accesos a BD. Generalmente tendremos una clase EntityManager por conexión a BD, en nuestro caso esto se traducirá en una por cada Bounded Context. Manejaremos estas conexiones desde el `EntityManagerFactory` y puesto que queremos customizar aspectos como dónde guardamos los xml de configuración, hemos creado nuestra propia factoría

Clase `MoocEntityManagerFactory`:

    final class MoocEntityManagerFactory
    {
        private const SCHEMA_PATH = __DIR__ . '/../../../../../databases/mooc.sql';
        public static function create(array $parameters, string $environment): EntityManagerInterface
        {
            $isDevMode = 'prod' !== $environment;
            $prefixes               = DoctrinePrefixesSearcher::inPath(__DIR__ . '/../../../../Mooc', 'CodelyTv\Mooc');
            $dbalCustomTypesClasses = DbalTypesSearcher::inPath(__DIR__ . '/../../../../Mooc', 'Mooc');
            return DoctrineEntityManagerFactory::create(
                $parameters,
                $prefixes,
                $isDevMode,
                self::SCHEMA_PATH,
                $dbalCustomTypesClasses
            );
        }
    }


Actualmente encontramos esta clase en el Shared del Bounded Context de Mooc, sin embargo podríamos optar por extraerla al Shared Kernel si una vez vayamos añadiendo mas factorías viésemos que la estructura es bastante similar. Cuando estemos en el entorno de Desarrollo, lo primero que hará será comprobar si existe la BD, creándola junto a las tablas que le especifiquemos en el fichero [mooc.sql](https://github.com/CodelyTV/php-ddd-skeleton/blob/0.13.0/databases/mooc.sql) si no existiera. Aquí es importante ver que el crear las tablas en base a unos inserts creados por nosotros se rige por ese principio de Outside-in 📥 que hemos venido siguiendo desde el inicio del curso

Dentro de la clase `DoctrinteEntityManagerFactory` comprobaremos el entorno en el que nos encontramos para saber si debemos crear de nuevo la BD, además registrará nuestros Custom Types y creará un EntityManager con la configuración que le estamos indicando. Por otro lado, dentro de nuestra configuración propia, le indicaremos que queremos utilizar el driver simplificado de xml de la gente de Symfony (Integrado en el ORM de Doctrine por defecto)

Para indicar que debe usar nuestra factoría cuando sea necesario inyectar un EntityManager en un repositorio hemos creado un servicio ‘factory’ que nos devolverá una instancia cuando llamemos al método _create_ de `MoocEntityManagerFactory`. A su vez, en esta clase hemos establecido la configuración necesaria para que escanee las carpetas indicadas (…/Infraestructure/Persistence/Doctrine/) y poder hacer así el mapeo con la carpeta Domain

¿Alguna Duda?
=============

Esta configuración es la que nos resulta más fácil para trabajar con Doctrine y la encontraréis en repo del curso para que podáis verla, utilizarla en vuestros proyectos o dejarnos PRs para dejarlo todo aún más fino 👌👌

Si tienes alguna duda sobre este video o quieres compartir alguna sugerencia no dudes en abrir una nueva discusión más abajo 👇👇👇

¡Nos vemos en el siguiente video: Mapeo de modelos en Doctrine Custom Types vs Embeddables!