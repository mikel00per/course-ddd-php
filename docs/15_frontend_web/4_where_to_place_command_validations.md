馃憖 Crear curso: D贸nde ubicar validaciones de comandos
=====================================================

Una de las premisas que venimos siguiendo en DDD es tener unos modelos de dominio ricos en comportamiento, empujando a ellos toda la l贸gica de dominio y haciendo que resida en ellos cualquier tipo de validaci贸n o restricci贸n de integridad que hayamos definido.

Sin embargo, la gesti贸n de los errores que ve铆amos en estos casos a trav茅s de clausulas de guarda y manejo de excepciones no es el tipo de gesti贸n que deber铆amos seguir para controlar aquellos errores que vengan de la interacci贸n de un usuario con el frontal de la aplicaci贸n. **A nivel de experiencia de usuario** no s贸lo queremos que se indique claramente qu茅 restricci贸n est谩 vulnerando en un campo, sino que tambi茅n deber铆amos **mostrar todos los errores de validaci贸n que est茅n produci茅ndose** en lugar de ir comprobando uno a uno (Recordemos que en el caso de las Validaciones de Dominio, en cuanto una de ellas saltaba, lanz谩bamos una excepci贸n que romp铆a con el flujo)

Una opci贸n para no dejar de hacerlo desde el Dominio podr铆a ser capturar cada una de estas excepciones mediante bloques try-catch para 鈥榓cumular鈥? estos errores de validaci贸n, sin embargo no nos convence mucho ya que por un lado implica un encadenamiento bastante 鈥榝eo鈥? try-catchs, y por otro lado ya nos estamos encontrando con campos en los que se pueden producir m煤ltiples errores (longitud m铆nima/m谩xima, caracteres obligatorios/no permitidos鈥?)

Como hemos venido hablando en varias ocasiones, **es mejor c贸digo duplicado que una mala abstracci贸n 鉁岋笍**, y en este caso apostamos por mantener por una parte las validaciones de Dominio tal y como estaban y por otra a帽adir en el controlador las validaciones que se reflejar谩n en la vista

Validando el Formulario desde el Controller 鉁旓笍
----------------------------------------------

Para la tarea de validar los campos del formulario nos apoyaremos en el componente 鈥榲alidator鈥? de symfony que nos brinda un formato bastante r谩pido y sencillo de definici贸n de las validaciones (Este componente lo utiliza el componente 鈥楩orm鈥? de Symfony por debajo)

    composer require symfony/validator


Clase `CoursesPostWebController`:

    final class CoursesPostWebController extends WebController
    {
        public function __invoke(Request $request)
        {
            $validationErrors = $this->validateRequest($request);
            return $validationErrors->count()
                ? $this->redirectWithErrors('courses_get', $validationErrors, $request)
                : $this->createCourse($request);
        }
        private function validateRequest(Request $request): ConstraintViolationListInterface
        {
            $constraint = new Assert\Collection(
                [
                    'id'       => new Assert\Uuid(),
                    'name'     => [new Assert\NotBlank(), new Assert\Length(['min' => 1, 'max' => 255])],
                    'duration' => [new Assert\NotBlank(), new Assert\Length(['min' => 4, 'max' => 100])],
                ]
            );
            $input = $request->request->all();
            return Validation::createValidator()->validate($input, $constraint);
        }
        private function createCourse(Request $request): RedirectResponse
        {
            $this->dispatch(
                new CreateCourseCommand(
                    $request->request->get('id'),
                    $request->request->get('name'),
                    $request->request->get('duration')
                )
            );
            return $this->redirectWithMessage(
                'courses_get',
                sprintf('Feliciades, el curso %s ha sido creado!', $request->request->get('name'))
            );
        }
    }


Cuando la petici贸n llega al Controlador lo primero que har谩 es validarla, de modo que si no hay errores seguir谩 el flujo normal de creaci贸n de curso, pero si hay erores nos devolver谩 a la vista anterior junto con los errores de validaci贸n detectados para poder pintarlos en la vista

La validaci贸n que hemos encapsulado en `validateRequest` se lleva a cabo pasando a cada campo la restricci贸n o array de restricciones que debe comprobar. Si finalmente esta validaci贸n devuelve alg煤n error, llamaremos al m茅todo `redirectWithErrors` que guardar谩 en mensajes flash de sesi贸n tanto los errores como los inputs que hab铆a introducido el usuario

Ya en el partial del formulario de creaci贸n de cursos (pod茅is ver la plantilla [aqu铆](https://github.com/CodelyTV/php-ddd-skeleton/blob/master/apps/backoffice/frontend/templates/pages/courses/partials/new_course_form.html.twig)) Hemos definido varias cl谩usulas 鈥榠f鈥? 馃暤

*   Si un input contiene errores pintaremos los bordes en rojo para resaltar el campo
*   Si un input contiene errores a帽adiremos un label debajo con la descripci贸n del error

驴Alguna Duda?
=============

Si tienes alguna duda sobre este video o quieres compartir alguna sugerencia no dudes en abrir una nueva discusi贸n m谩s abajo 馃憞馃憞馃憞

隆Nos vemos en la siguiente lecci贸n: 馃О Backoffice!