Implementación del endpoint y test de aceptación
================================================

Siguiendo con el ejemplo de la plataforma de CodelyTV implementaremos el caso de uso de **Crear un nuevo Curso**, sobre el que iteraremos en los siguientes videos. Puesto que queremos mantener el flujo outside-in, lo primero que debemos hacer es crear el test de aceptación para dicho caso de uso

Añadiendo el test de aceptación ✅
---------------------------------

Para empezar crearemos un nuevo módulo en el directorio _test/_ para este nuevo recurso (Cursos) y dentro de éste escribiremos la [feature](https://github.com/CodelyTV/php-ddd-skeleton/blob/0.6.0/tests/apps/mooc/backend/features/courses/course_put.feature) para el caso de uso `course_put`

feature `course_put`:

    Feature: Create a new course
      In order to have courses on the platform
      As a user with admin permissions
      I want to create a new course
    
      Scenario: A valid non existing course
        Given I send a PUT request to "/courses/1aab45ba-3c7a-4344-8936-78466eca77fa" with body:
        """
        {
          "name": "The best course",
          "duration": "5 hours"
        }
        """
        Then the response status code should be 201
        And the response should be empty


Tal como ya vimos en el curso de [cqrs](https://pro.codely.tv/library/cqrs-command-query-responsibility-segregation-3719e4aa/62554/about/), queremos que **el identificador del nuevo recurso lo defina el cliente**, por eso la petición será PUT en lugar de POST. De este modo:

*   👆El cliente ya conoce el identificador y no espera una respuesta con éste por lo que el servidor simplemente tendrá que devolver un `status 201` si todo ha ido bien
*   ✌️Conseguimos que no se violen las reglas de integridad de nuestro dominio al poder especificar ese id desde el propio cliente en el momento en que hacemos un `new` del recurso

Si lanzamos en este punto los test, veremos que simplemente no se ejecuta ya que aún no está registrado, para ello añadiremos una nueva suite en el fichero `mooc_backend.yml` con el path donde se encuentran las features de este recurso

## Creando el endpoint 📥

Dentro del directorio _config/routes/_ del backend de nuestra aplicación crearemos un nuevo fichero yaml como el que definimos para el caso de uso de health-check, pero con la diferencia de que en este caso le pasaremos el parámetro del identificador en el path y definiremos el método http PUT

Con nuestra ruta preparada, sólo nos queda definir el [controlador](https://github.com/CodelyTV/php-ddd-skeleton/blob/0.6.0/apps/mooc/backend/src/Controller/Courses/CoursesPutController.php) que recogerá la petición y retornará un status 201

Clase `CoursesPutController`:

    use Symfony\Component\HttpFoundation\Request;
    use Symfony\Component\HttpFoundation\Response;
    final class CoursesPutController
    {
        public function __invoke(string $id, Request $request)
        {
            return new Response('', 201);
        }
    }


La magia interna de Symfony nos facilita recibir como parámetro en el controlador la variable que se haya definido en el path de la ruta (Ojo 👀 que nos llegará como un string)

¿Alguna Duda?
=============

Si tienes alguna duda sobre el contenido del video o quieres compartir alguna sugerencia no dudes en abrir una nueva discusión más abajo 👇👇👇

¡Nos vemos en el siguiente video: Integrar PHPUnit para tests unitarios!