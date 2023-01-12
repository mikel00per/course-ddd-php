👀 Crear curso: Dónde ubicar validaciones de comandos
=====================================================

Una de las premisas que venimos siguiendo en DDD es tener unos modelos de dominio ricos en comportamiento, empujando a ellos toda la lógica de dominio y haciendo que resida en ellos cualquier tipo de validación o restricción de integridad que hayamos definido.

Sin embargo, la gestión de los errores que veíamos en estos casos a través de clausulas de guarda y manejo de excepciones no es el tipo de gestión que deberíamos seguir para controlar aquellos errores que vengan de la interacción de un usuario con el frontal de la aplicación. **A nivel de experiencia de usuario** no sólo queremos que se indique claramente qué restricción está vulnerando en un campo, sino que también deberíamos **mostrar todos los errores de validación que estén produciéndose** en lugar de ir comprobando uno a uno (Recordemos que en el caso de las Validaciones de Dominio, en cuanto una de ellas saltaba, lanzábamos una excepción que rompía con el flujo)

Una opción para no dejar de hacerlo desde el Dominio podría ser capturar cada una de estas excepciones mediante bloques try-catch para ‘acumular’ estos errores de validación, sin embargo no nos convence mucho ya que por un lado implica un encadenamiento bastante ‘feo’ try-catchs, y por otro lado ya nos estamos encontrando con campos en los que se pueden producir múltiples errores (longitud mínima/máxima, caracteres obligatorios/no permitidos…)

Como hemos venido hablando en varias ocasiones, **es mejor código duplicado que una mala abstracción ✌️**, y en este caso apostamos por mantener por una parte las validaciones de Dominio tal y como estaban y por otra añadir en el controlador las validaciones que se reflejarán en la vista

Validando el Formulario desde el Controller ✔️
----------------------------------------------

Para la tarea de validar los campos del formulario nos apoyaremos en el componente ‘validator’ de symfony que nos brinda un formato bastante rápido y sencillo de definición de las validaciones (Este componente lo utiliza el componente ‘Form’ de Symfony por debajo)

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


Cuando la petición llega al Controlador lo primero que hará es validarla, de modo que si no hay errores seguirá el flujo normal de creación de curso, pero si hay erores nos devolverá a la vista anterior junto con los errores de validación detectados para poder pintarlos en la vista

La validación que hemos encapsulado en `validateRequest` se lleva a cabo pasando a cada campo la restricción o array de restricciones que debe comprobar. Si finalmente esta validación devuelve algún error, llamaremos al método `redirectWithErrors` que guardará en mensajes flash de sesión tanto los errores como los inputs que había introducido el usuario

Ya en el partial del formulario de creación de cursos (podéis ver la plantilla [aquí](https://github.com/CodelyTV/php-ddd-skeleton/blob/master/apps/backoffice/frontend/templates/pages/courses/partials/new_course_form.html.twig)) Hemos definido varias cláusulas ‘if’ 🕵

*   Si un input contiene errores pintaremos los bordes en rojo para resaltar el campo
*   Si un input contiene errores añadiremos un label debajo con la descripción del error

¿Alguna Duda?
=============

Si tienes alguna duda sobre este video o quieres compartir alguna sugerencia no dudes en abrir una nueva discusión más abajo 👇👇👇

¡Nos vemos en la siguiente lección: 🧰 Backoffice!