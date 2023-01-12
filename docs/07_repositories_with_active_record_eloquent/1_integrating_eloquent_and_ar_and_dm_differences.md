Integrando Eloquent y diferencias entre Active Record y Data Mapper
===================================================================

Para trabajar con Active Record, como en el caso de Eloquent, necesitaremos crear una clase que extienda de `Model`, lo cual nos proveerá de un conjunto de métodos que realizarán su propia magia de forma interna

*   Podéis encontrar el código relativo a este video en el tag [0.18.0 del repo](https://github.com/CodelyTV/php-ddd-skeleton/tree/0.18.0)

Clase `EloquentCourseRepository`:

    final class EloquentCourseRepository implements CourseRepository
    {
        // ...
        
        public function search(CourseId $id): ?Course
        {
            $model = CourseEloquentModel::find($id->value());
            if (null === $model) {
                return null;
            }
            return new Course(new CourseId($model->id), new CourseName($model->name), new CourseDuration($model->duration));
        }
    }


Un ejemplo de estos métodos será _find_, que a partir del `id` que le pasemos nos devolverá un modelo con los mismos atributos que tenga en BD. Estos métodos no dejan de ser [named constructor](https://codely.tv/blog/screencasts/constructores-semanticos/) que devuelven una instancia de la clase desde la cual se está llamando (En este caso `CourseEloquentModel`). Vamos a ver pasito a pasito cómo evitar que nuestro Dominio acabe contaminado por las queries a BD y el resto de magias que tiene lugar en estos named constructors 🧹

Instalando Eloquent
-------------------

    composer require wouterj/eloquent-bundle


Este paquete nos permitirá integrar el ORM que de base no es nativo para Symfony sin que tengamos que picar nosotros mismos toda esta adaptación. Tal como venimos haciendo con todas las dependencias externas, la añadiremos desde la línea de comandos. Además, tendremos que añadirlo dentro del fichero [bundles.php](https://github.com/CodelyTV/php-ddd-skeleton/blob/0.14.0/apps/mooc/backend/config/bundles.php)

Fichero `services_test`:

    wouterj_eloquent:
      driver:   mysql
      host:     '%env(MOOC_DATABASE_HOST)%'
      port:     '%env(MOOC_DATABASE_PORT)%'
      database: '%env(MOOC_DATABASE_NAME)%'
      username: '%env(MOOC_DATABASE_USER)%'
      password: '%env(MOOC_DATABASE_PASSWORD)%'
      prefix:   ~
      eloquent: ~


El último elemento de configuración que necesitaremos será añadir en nuestro ‘services\_test.yaml’ la configuración que el bundle requiere. Es importante que añadamos esta última línea para que cargue Eloquent (también es la pieza que puede responderá de formas ‘raras’ ante conflictos en la inyección de dependencias)

¿Alguna Duda?
=============

Si tienes alguna duda sobre este video o quieres compartir alguna sugerencia no dudes en abrir una nueva discusión más abajo 👇👇👇

¡Nos vemos en el siguiente video: Repositorios con Eloquent: Escondiendo en la infraestructura la contaminación!