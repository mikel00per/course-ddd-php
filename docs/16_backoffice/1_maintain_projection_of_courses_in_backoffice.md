🚦 Listado de cursos: Mantener proyección de cursos en Backoffice
=================================================================

Ya que tenemos controlada la creación de nuevos Cursos desde nuestro backoffice, lo que vamos a hacer es listar todos los Cursos creados sin necesidad de atacar al contexto de Mooc, es decir, **el contexto de backoffice tendrá su propia proyección de Cursos** a la que podremos lanzar nuestras queries de consulta

Pintando los registros en la template 🖌
----------------------------------------

A nivel de cliente, si analizamos el partial correspondiente al [listado de cursos](https://github.com/CodelyTV/php-ddd-skeleton/blob/master/apps/backoffice/frontend/templates/pages/courses/partials/list_courses.html.twig) veremos que desde Javascript la función `addCoursesList()` realiza un **fetch a un controlador del frontal de backoffice** y rellena la tabla con los datos que le devuelve la llamada

Clase `CoursesGetController`:

    final class CoursesGetController extends Controller
    {
    
        public function __invoke(Request $request): JsonResponse
        {
            /** @var BackofficeCoursesResponse $response */
            $response = $this->queryBus->ask(new SearchAllBackofficeCoursesQuery()
            );
            return new JsonResponse(map($this->toArray(), $response->courses()));
        }
    
        private function toArray(): callable
        {
            return static function (BackofficeCourseResponse $course) {
                return [
                    'id'       => $course->id(),
                    'name'     => $course->name(),
                    'duration' => $course->duration(),
                ];
            };
        }
    }


Desde el controlador lo que haremos será lanzar una `SearchAllBackofficeCoursesQuery` al bus de backoffice para recuperar todos los cursos y devolveremos la respuesta en formato Json (mapeamos la respuesta a un array con la estructura que nos interesa y lo parseamos al devolverlo)

Para ‘llenar’ la proyección de Cursos a la cual estamos consultando lo que haremos será definir un subscriber que escuche el famoso `CourseCreatedDomainEvent` que lo único que hará será crear el agregado de este curso y guardarlo en la tabla de proyección (Actualmente esta tabla está dentro de la infraestructura de Mooc, pero sería considerablemente fácil sacar estas tablas de backoffice a una BD propia)

Puesto que la petición ‘fetch’ desde Javascript la realiza cada vez que se carga la página, cuando se envíe el Post de crear curso y vuelva a redirigirnos a esta misma vista, estaremos recuperando también este nuevo curso que ya se habrá proyectado en nuestra tabla de backoffice

¿Alguna Duda?
=============

Si tienes alguna duda sobre este video o quieres compartir alguna sugerencia no dudes en abrir una nueva discusión más abajo 👇👇👇

¡Nos vemos en el siguiente video: 🚁 Filtrado de elementos: Patrón Criteria/Specification!