⑩ Contador de cursos vía query desde Controller
===============================================

Ya empieza a tomar forma nuestro backoffice y hemos añadido una sección ‘Cursos’ donde queremos mostrar el total de cursos creados actualmente, para recuperar este dato tendremos que consultar al contexto de Mooc donde ya teníamos una query que hacía esta tarea (más adelante veremos la opción de tenerlo en la BD de backoffice) y esto lo haremos desde el controlador de esta vista

Siguiendo con el sistema de plantillas que iniciamos en el video anterior hemos definido una plantilla ‘master’ que además de incluir los partials del header y el footer de la web espera recibir un bloque ‘main’ con el que podremos jugar de tal modo que cualquier sección pueda utilizar esta plantilla maestra preocupándose sólo implementar este ‘main’

Si vemos la [plantilla de cursos](https://github.com/CodelyTV/php-ddd-skeleton/blob/master/apps/backoffice/frontend/templates/pages/courses/courses.html.twig) encontramos que ésta espera cargar dinámicamente el contador de cursos, que tendremos que pasarle desde el controlador ⑩

Modificando el Controller 🛠
----------------------------

Clase `CoursesGetWebController` (en el video CoursesGetController):

    final class CoursesGetWebController extends WebController
    {
     
        public function __invoke(Request $request): Response
        {
            /** @var CoursesCounterResponse $coursesCounterResponse */
            $coursesCounterResponse = $this->ask(new FindCoursesCounterQuery());
     
            return $this->render(
                'pages/courses/courses.html.twig',
                [
                    'title'           => 'Courses',
                    'description'     => 'Courses CodelyTV - Backoffice',
                    'courses_counter' => $coursesCounterResponse->total(),
                ]
            );
        }
    }


*   💡 Se ha modificado el nombre de los controladores para diferenciar claramente que estos están orientados a renderizar el frontal web del backoffice a diferencia de los controladores con los que hemos trabajado en el contexto de Mooc

Un primer detalle es que hemos extraído a la [clase abstracta](https://github.com/CodelyTV/php-ddd-skeleton/blob/master/src/Shared/Infrastructure/Symfony/WebController.php) `WebController` toda esa lógica común entre los distintos controladores (devolver el renderizado con twig, llamadas al QueryBus para solicitar contenido dinámico, enviar contenido vía CommandBus…), este será uno de los pocos casos en los que nos interese utilizar la herencia entre clases (utilizar la composición en este caso llega a ser muy verboso y además tenemos claro que sólo habrá un nivel único de herencia)

El método _ask()_ que utilizamos desde el controlador (Podéis ver la implementación [aquí](https://github.com/CodelyTV/php-ddd-skeleton/blob/master/src/Shared/Infrastructure/Symfony/ApiController.php#L34)) espera recibir una Query del tipo que sea y, en nuestro caso le pasaremos una `FindCoursesCounterQuery` que como vimos está definida justamente en el contexto de Mooc. El QueryBus nos devolverá una CoursesCounterResponse, que al ser un DTO con los datos en primitivo podremos pasarle el valor directamente sin tener que llamar a ningún ‘_value()_’ como en el caso de los Value Objects. Otra consideración de este ask() es que está abierto a que la búsqueda no nos devuelva ningún resultado y lo que retornemos sea directamente un `null`

Si queremos lanzar una prueba en nuestro local para ver si todo está en orden hasta este punto, es importante que editemos el fichero ‘.env’ donde se encuentra la configuración para la conexión BD. A la hora de levantar este entorno podemos ejecutar el [script](https://github.com/CodelyTV/php-ddd-skeleton/blob/master/databases/mooc.sql) de SQL, aunque si ejecutamos los tests de aceptación, estos ya harán internamente este proceso

*   Ojo 👀!! Modificar la BD directamente es algo que debemos evitar a toda costa. Estamos trabajando con Eventos de Dominio y estos son nuestro ‘source of truth’, por lo que cualquier cambio manual en BD que hagamos no se estará reflejando en los eventos y cualquiera que los importe para tener el estado actual, no tendrá los mismos valores que nosotros y no estará funcionando correctamente (Friendly reminder: ⚠️ No modifiquemos la BD a mano ⚠️)

¿Alguna Duda?
=============

Si tienes alguna duda sobre este video o quieres compartir alguna sugerencia no dudes en abrir una nueva discusión más abajo 👇👇👇

¡Nos vemos en el siguiente video: 🎓 Crear curso: Formul