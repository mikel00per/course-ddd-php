🎓 Crear curso: Formulario y command desde Controller
=====================================================

Ya hemos visto cómo renderizamos dinámicamente el contador de cursos en el template a través de una Query al contexto de Mooc, así que el siguiente paso será poder crear nuevos cursos desde un formulario 📝 y ver cómo esta acción deriva en en la actualización de dicho contador

El proceso que seguiremos para esto pasará por recoger los campos del formulario en el controlador y desde ahí enviar un Comando al contexto de Mooc que como acción principal nos registrará este nuevo curso y además, vía subscriptor que escucha el evento de `CourseCreated`, se llamará al caso de uso que incrementa el contador del total de cursos

Clase `CoursesGetWebController`:

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
                    'new_course_id'   => Uuid::random()->value(),
                ]
            );
        }
    }


En primer lugar haremos que el identificador del curso sea un valor generado desde el cliente. Para ello nos valdremos de nuestra clase `Uuid` que teníamos en el Shared Kernel y que nos permitirá generar un uuid aleatorio cada vez que recarguemos la vista (Fuera del formato explicativo que tiene este formulario, haríamos que el input del identificador fuera de tipo ‘hidden’ ya que de cara al usuario sería innecesario)

Además de autogenerar el Uuid, necesitaremos establecer la ruta de envío del formulario en el fichero de configuración. Puesto que estamos enviando el recurso desde HTML, tendremos que utilizar el método POST (a diferencia de nuestra API en el backend de Mooc donde podíamos utilizar el método PUT) a pesar de estár pasándole el identificador del propio recurso

Clase `CoursesPostWebController`:

    final class CoursesPostWebController extends WebController
    {
    
        public function __invoke(Request $request)
    
            $command = new CreateCourseCommand(
                    $request->request->get('id'),
                    $request->request->get('name'),
                    $request->request->get('duration')
                )
    
            $this->dispatch($command);
    
            return $this->redirect('/courses');
        }
    }


Desde el controlador para la petición POST lo que haremos será montar un nuevo `CreateCourseCommand` con los campos recibidos en la Request (esta Request no es custom sino del propio framework), recogiéndolos por el ‘name’ de los inputs del formulario. Una vez enviado el comando redireccionaremos a la vista previa, recordemos que el método POST nos envía a otra ruta (otra alternativa podría ser redireccionar a la vista de edición del recurso recien creado)

Si queremos probarlo en nuestro entorno local es aconsejable indicar que usaremos el EventBus en memoria en lugar de RabbitMQ, Ojo 👀 porque esta configuración la definimos en el `services_test` porque en nuestro fichero de Enviroment (.env) le hemos establecido que nuestro entorno (APP\_ENV) es el de test

¿Alguna Duda?
=============

Si tienes alguna duda sobre este video o quieres compartir alguna sugerencia no dudes en abrir una nueva discusión más abajo 👇👇👇

¡Nos vemos en el siguiente video: 👀 Crear curso: Dónde ubicar validaciones de comandos!