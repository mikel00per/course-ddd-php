🤷‍♀️ Testeando guardar en el repositorio con eventos de dominio en el agregado
===============================================================================

Ahora que hemos añadido Mockery, vamos a dar otro pasito mas en nuestros tests añadiendo Similar Comparators y vamos a verlo con el propio caso de uso de inicializar un nuevo contador cuando recibimos un evento de dominio de tipo ‘video creado’ y guardarlo en BD

*   Podéis encontrar el código relativo a este video en el tag [0.17.0 del repo](https://github.com/CodelyTV/php-ddd-skeleton/tree/0.17.0)

Clase `IncrementCoursesCounterOnCourseCreatedTest`:

    final class IncrementCoursesCounterOnCourseCreatedTest extends CoursesCounterModuleUnitTestCase
    {
        private $subscriber;
    
        protected function setUp(): void
        {
            parent::setUp();
            $this->subscriber = new IncrementCoursesCounterOnCourseCreated(
                new CoursesCounterIncrementer(
                    $this->repository(),
                    $this->uuidGenerator(),
                    $this->domainEventPublisher()
                )
            );
        }
        
        /** @test */
        public function it_should_initialize_a_new_counter(): void
        {
            $event = CourseCreatedDomainEventMother::random();
            $courseId    = CourseIdMother::create($event->aggregateId());
            $newCounter  = CoursesCounterMother::withOne($courseId);
            $domainEvent = CoursesCounterIncrementedDomainEventMother::fromCounter($newCounter);
            $this->shouldSearch(null);
            $this->shouldGenerateUuid($newCounter->id()->value());
            $this->shouldSave($newCounter);
            $this->shouldPublishDomainEvent($domainEvent);
            $this->notify($event, $this->subscriber);
        }
        // ...
    }


Estamos creando un nuevo `CoursesCounter` a partir del named constructor [withOne](https://github.com/CodelyTV/php-ddd-skeleton/blob/0.17.0/tests/src/Mooc/CoursesCounter/Domain/CoursesCounterMother.php#L24) del `CoursesCounterMother`, en el que le indicando que lo inicialice con 1 curso, pasándole precisamente el `courseId` que hemos recibido del evento recien generado desde su correspondiente ObjectMother.

Por otra parte, en tiempo de tests no estamos definiendo el atributo _domainEvent_ que `CoursesCounter` hereda de `AggregateRoot`, por lo que al llamar al método _shouldSave()_ debíamos, hasta ahora, indicarle únicamente que debería invocarse al _save()_ del repositorio con algún parámetro ya que ese atributo no coincidiría 💔

Para solucionar este problema y testear mejor nuestra aplicación, lo primero que haremos será sustituir el método _shouldSave()_ de nuestra clase auxiliar

Método `CoursesCounterModuleUnitTestCase->shouldSave($course)` antes:

    protected function shouldSave(CoursesCounter $course): void
    {
        $this->repository()->method('save')->withAnyParameters();
    }


Método `CoursesCounterModuleUnitTestCase->shouldSave($course)` ahora:

    protected function shouldSave(CoursesCounter $course): void
    {
        $this->repository()
            ->shouldReceive('save')
            ->once()
            ->with($this->similarTo($course))
            ->andReturnNull();
    }


_similarTo()_ es un método que hemos prepadado en la clase `UnitTestCase` (y recogida a su vez en `TestUtils`) aprovechando los **Matchers de PHPUnit** que nos permita comprobar que todos los datos de ambas clases sean iguales salvo el atributo _domainEvents_ 🕵. A su vez, este método lo que devolverá será un `CodelyTvMatcherIsSimilar` que hereda del `MatcherAbstract` de Mockery

Si analizamos este CodelyTvMatcherIsSimilar veremos cómo en el constructor está instanciando una `CodelyTvConstraintIsSimilar`, la cual hereda de la clase `Constraint` de PHPUnit y es la que condensa toda la magia de nuestra comparación custom como podéis ver [aquí](https://github.com/CodelyTV/php-ddd-skeleton/blob/0.17.0/tests/src/Shared/Infrastructure/PhpUnit/Constraint/CodelyTvConstraintIsSimilar.php)

Resumidamente estamos recuperando el `IsEqual` de PHPUnit para modificarlo en base a nuestras necesidades. En primer lugar hemos [registrado](https://github.com/CodelyTV/php-ddd-skeleton/blob/0.17.0/tests/src/Shared/Infrastructure/PhpUnit/Constraint/CodelyTvConstraintIsSimilar.php#L39-L45) todos los comparadores customs que vamos a utilizar, de tal modo que entrará en uno u otro en función del tipo de instancia que le estemos pasando

Si entramos en el comparador del AggregateRoot, podemos comprobar que lo que estamos haciendo es borrar los eventos de dominio del Agregado actual y entonces compararlo, de modo que este campo no conlleve ninguna diferencia entre ambas instancias

Cuando lo que queramos testear que se ha publicado un evento de dominio tendremos la misma casuística y, en este caso, entrará en el `DomainEventSimilarComparator` donde le especificaremos que ignore no sólo el id del evento sino el _occurred\_on_, puesto que claramente diferirán siempre en ambos campos

¿Alguna Duda?
=============

Si tienes alguna duda sobre este video o quieres compartir alguna sugerencia no dudes en abrir una nueva discusión más abajo 👇👇👇

¡Nos vemos en el siguiente video: 🤔 Alternativa: Publicar eventos desde agregados!