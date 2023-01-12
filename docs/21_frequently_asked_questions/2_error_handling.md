🤕 Gestión de errores
======================

Hasta ahora habíamos visto que para la **gestión de errores a nivel de comunicación HTTP** hacíamos uso de distintos bloques try-catch donde establecíamos el tipo de Response que devolvíamos. Esta forma de hacerlo, además de ‘ensuciar’ bastante nuestros Controllers con lógica duplicada, nos aporta muy poco valor, por lo que daremos un paso más en este aspecto centralizando la gestión de estos errores en un único punto común

Para ello, lo primero que haremos será añadir en los Controllers de nuesta API HTTP la clase `ApiController` (Bien de forma directa o a través del `WebController`) que será precisamente ese punto en común y que recibe además del CommandBus y el QueryBus la clase `ApiExceptionsHttpStatusCodeMapping`. Este ApiController nos obliga a implementar el método _exceptions()_ desde el cual definimos el mapeo de Excepción-Respuesta propio de cada Controller que se añadirán al conjunto de excepciones ‘comunes’

Pero para que se produzca la magia sin que los Controllers tengan que ocuparse de nada hace falta que añadamos otra pieza: un EventListener de Symfony (Como ya vimos a la hora de crear middlewares para la autenticación) que escuche cuando se produce un evento de tipo exception para llamar a la clase `ApiExceptionListener` (establecido en el fichero de definición de servicios)

Clase `ApiExceptionListener`:

    final class ApiExceptionListener
    {
        private $exceptionHandler;
    
        public function __construct(ApiExceptionsHttpStatusCodeMapping $exceptionHandler)
        {
            $this->exceptionHandler = $exceptionHandler;
        }
    
        public function onException(RequestEvent $event): void
        {
            $exception = $event->getException();
    
            $event->setResponse(
                new JsonResponse(
                    [
                        'code'    => $this->exceptionCodeFor($exception),
                        'message' => $exception->getMessage(),
                    ],
                    $this->exceptionHandler->statusCodeFor(get_class($exception))
                )
            );
        }
        private function exceptionCodeFor(Exception $error)
        {
            $domainErrorClass = DomainError::class;
            return $error instanceof $domainErrorClass ? $error->errorCode() : Utils::toSnakeCase(class_basename($error));
        }
    }


El listener recibe por constructor el mapping de excepciones, que es la misma instancia que se había inicializado en el constructor del Controller. Cuando le llega el evento, lo que hace es recuperar la excepción que contiene y con ella sacar el código de error que se enviará en la respuesta, en caso de no encontrar un mapeo para la excepción recibida lo que haremos será pasarle por defecto un ‘Error 500’

Si quisieramos registrar todas las excepciones que se produzcan, lo que haríamos es definir otro EventListener que actue antes de este `ApiExceptionListener` recuperando la excepción lanzada y loggeandola o almacenándola para nuestras analíticas 📈

¿Alguna Duda?
=============

Si tienes alguna duda sobre este video o quieres compartir alguna sugerencia no dudes en abrir una nueva discusión más abajo 👇👇👇

¡Nos vemos en el siguiente Video: 😮 Casos de uso complejos que requieren información de varios contextos!