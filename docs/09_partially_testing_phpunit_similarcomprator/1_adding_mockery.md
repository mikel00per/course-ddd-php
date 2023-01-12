🤡 Añadiendo Mockery
====================

Tal como veíamos en el video anterior, los mocks que nos provee PHPUnit no nos permiten comparar parcialmente los atributos de clase entre dos instancias de una misma clase, lo cual supone un problema a la hora de desarrollar los tests para nuestros casos de uso. Para solventar esta limitación lo que haremos será utilizar [Mockery](http://docs.mockery.io/en/latest/)

*   Podéis encontrar el código relativo a este video en el tag [0.17.0 del repo](https://github.com/CodelyTV/php-ddd-skeleton/tree/0.17.0)

Puesto que sólo queremos utilizar esta librería para nuestros tests, añadiremos la dependencia con la cláusula ‘–dev’

    composer require --dev mockery/mockery


Clase `UnitTestCase`:

    abstract class UnitTestCase extends MockeryTestCase
    {
        private $domainEventPublisher;
        private $uuidGenerator;
    
    protected function mock(string $className): MockInterface
        {
            return Mockery::mock($className);
        }
    
    protected function shouldPublishDomainEvent(DomainEvent $domainEvent): void
        {
            $this->domainEventPublisher()
                ->shouldReceive('publish')
                ->with($this->similarTo($domainEvent))
                ->andReturnNull();
        }
    
    /** @return DomainEventPublisher|MockInterface */
        protected function domainEventPublisher(): MockInterface
        {
            return $this->domainEventPublisher = $this->domainEventPublisher ?: $this->mock(DomainEventPublisher::class);
        }
    
        /** @return UuidGenerator|MockInterface */
        protected function uuidGenerator(): MockInterface
        {
            return $this->uuidGenerator = $this->uuidGenerator ?: $this->mock(UuidGenerator::class);
        }
    
        // ...
    }


La primera diferencia que podemos encontrar ahora es que ahora, en lugar de heredar del `TestCase` de PHPUnit, lo haremos de `MockeryTestCase`, lo cual nos aportará múltiples utilidades para la creación y control de los Mocks 🧰

También añadimos una función propia _mock()_ que internamente llamará al método con mismo nombre de `Mockery` Podemos ver cómo este método _mock()_ ya está siendo utilizado en el método _uuidGenerator()_. Aquí encontramos otra diferencia y es que ahora el tipo de retorno no es un `MockObject`sino un `MockInterface` de Mockery

A nivel de paso de mensajes también existen algunas diferencias: si vemos el método _shouldPublishDomainEvent()_ lo que estaremos indicando es que la clase `uuidGenerator` debería estar recibiendo el mensaje ‘generate’, sólo una vez, sin argumentos y devolver el uuid que le estamos pasando

¿Alguna Duda?
=============

Si tienes alguna duda sobre este video o quieres compartir alguna sugerencia no dudes en abrir una nueva discusión más abajo 👇👇👇

¡Nos vemos en el siguiente video: 🤷‍♀️ Testeando guardar en el repositorio con eventos de dominio en el agregado!