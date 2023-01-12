🥁 Integración de Symfony Messenger como Command Bus
====================================================

Llegados a esta lección os recomendamos encarecidamente que hayáis pasado previamente por el curso de [CQRS](https://pro.codely.tv/library/cqrs-command-query-responsibility-segregation-3719e4aa/62554/about/) donde entramos en mayor profundidad en cada detalle sus particularidades y qué beneficios supone dar este paso (especialmente trabajando con Arquitectura Hexagonal)

Clase `InMemorySymfonyCommandBus` (Resultado final):

    final class InMemorySymfonyCommandBus implements CommandBus
    {
        private $bus;
        public function __construct(iterable $commandHandlers)
        {
            $this->bus = new MessageBus(
                [
                    new HandleMessageMiddleware(
                        new HandlersLocator(CallableFirstParameterExtractor::forCallables($commandHandlers))
                    ),
                ]
            );
        }
        public function dispatch(Command $command): void
        {
            try {
                $this->bus->dispatch($command);
            } catch (NoHandlerForMessageException $unused) {
                throw new CommandNotRegisteredError($command);
            } catch (HandlerFailedException $error) {
                throw $error->getPrevious();
            }
        }
    }


Ya habíamos utilizado **Symfony Messenger** para el Bus de eventos de dominio en local, por lo que aprovecharlo para nuestros comandos no supondrá mayor dificultad. La diferencia radicará en que, mientras que 1 evento podía ir a N subscribers, los comandos tienen siempre una relación 1 a 1 con su CommandHandler

Tal como hicimos con la inyección de subscribers en la gestión de eventos de dominio, estamos definiendo en el fichero **services.yaml** que toda clase que implemente la interface `CommandHandler` le añadiremos el tag _codely.command\_handler_ para posteriormente inyectar en el `InMemorySymfonyCommandBus` toda clase con dicho tag

Desde el CommandBus volveremos a utilizar el mismo Middleware que ya vimos en el caso de `InMemorySymfonyEventBus`. Sin embargo, a diferencia del caso de los eventos, en esta ocasión nuestra interface no está obligando a implementar ningún método por contrato, simplemente la utilizamos para la inyección de los CommandHandlers (Hemos apostado por esta opción, pero si abogas mas por otra alternativa nos encantaría que la compartas abriendo una nueva discusión abajo! 👇👇)

A pesar de que el HandlersLocator espera un array como valor para cada clave (recordemos que recibiría un array multidimensional)el mapeo será una relación de 1 Command por cada CommandHandler

Finalmente y tal como vimos en el curso de CQRS, el método _dispatch()_ no tiene ningún tipo de retorno para asegurarnos de que los comandos no tienen respuesta (vendrá todo desde fuera), si que lanzaremos una `CommandNotRegisteredError` en el caso de no encontrar un CommandHandler asociado, ya que a diferencia del EventBus en el que asumíamos que podía haber casos en los que nadie estuviera aún subscrito a un determinado Evento, aquí partimos de la premisa de que la ejecución de un Comando es algo intencional y el hecho de no tener un CommandHandler asociado si que implica un problema (Se ha añadido también la captura de una posible `HandlerFailedException`)

¿Alguna Duda?
=============

Si tienes alguna duda sobre este video o quieres compartir alguna sugerencia no dudes en abrir una nueva discusión más abajo 👇👇👇

¡Nos vemos en el siguiente video: ♻️ Refactoring de Arquitectura Hexagonal a CQRS!