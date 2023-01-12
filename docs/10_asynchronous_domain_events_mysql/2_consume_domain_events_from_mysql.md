👾 Consumir eventos de dominio desde MySQL
==========================================

Ahora que sabemos cómo publicar los eventos de dominio, guardándolos en este caso en nuestra BD MySQL, veremos el siguiente paso que consiste en crear el consumidor que los recupere para que sean recibidos por sus correspondientes subscriptores

*   Podéis encontrar el código relativo a este video en la [rama principal del repo](https://github.com/CodelyTV/php-ddd-skeleton/)

El consumidor se situa en la capa de infraestructura, ya que no sólo estamos tocando Entrada/Salida de datos, sino que nos estamos acoplando fuertemente a la implementación concreta de MySQL. No definiremos ninguna interface ya que las estrategias de consumo que se aplican para este caso, y por ende sus particularidades, hacen que no merezca la pena. Además, al no estar inyectando el consumidor en ningún caso de uso como colaborador, no tendremos necesidad de desacoplarnos de él para testarlo

Clase `MySqlDoctrineDomainEventsConsumer`:

    final class MySqlDoctrineDomainEventsConsumer
    {
        private $connection;
        private $eventMapping;
        public function __construct(EntityManager $entityManager, DomainEventMapping $eventMapping)
        {
            $this->connection   = $entityManager->getConnection();
            $this->eventMapping = $eventMapping;
        }
        public function consume(callable $subscribers, int $eventsToConsume): void
        {
            $events = $this->connection
                ->executeQuery("SELECT * FROM domain_events ORDER BY occurred_on ASC LIMIT $eventsToConsume")
                ->fetchAll(FetchMode::ASSOCIATIVE);
            each($this->executeSubscribers($subscribers), $events);
            $ids = implode(', ', map($this->idExtractor(), $events));
            if (!empty($ids)) {
                $this->connection->executeUpdate("DELETE FROM domain_events WHERE id IN ($ids)");
            }
        }
        private function executeSubscribers(callable $subscribers): callable
        {
            return function (array $rawEvent) use ($subscribers): void {
                try {
                    $domainEventClass = $this->eventMapping->for($rawEvent['name']);
                    $domainEvent      = $domainEventClass::fromPrimitives(
                        $rawEvent['aggregate_id'],
                        Utils::jsonDecode($rawEvent['body']),
                        $rawEvent['id'],
                        $this->formatDate($rawEvent['occurred_on'])
                    );
                    $subscribers($domainEvent);
                } catch (\RuntimeException $error) {
                }
            };
        }
        private function formatDate($stringDate): string
        {
            return Utils::dateToString(new DateTimeImmutable($stringDate));
        }
        private function idExtractor(): callable
        {
            return static function (array $event): string {
                return "'${event['id']}'";
            };
        }
    }


Pasaremos por constructor el EntityManager para establecer la conexión a BD y el [DomainEventMapping](https://github.com/CodelyTV/php-ddd-skeleton/blob/master/src/Shared/Infrastructure/Bus/Event/DomainEventMapping.php) desde el que establecemos el mapeo entre el nombre del evento y su clase. Para realizar este mapeo aprovechamos que hemos taggeado todos los subscribers para extraer los eventos que está escuchando 👂 (Aunque tal como se plantea, si recuperamos un evento al que no se haya subscrito aún nadie, nos dará error)

Dentro de nuestro consumidor el método _consume()_ será la pieza central de esta fase, este método espera recibir un callable con los subscriptores así como el número de eventos a consumir. Dentro del método, una vez recuperados los eventos ordenados por orden de publicación desde BD, ejecutaremos para cada subscriptor una llamada a _executeSubscribers()_ 📥

Esta función _executeSubscribers()_ lo primero que intentará (por eso lo recogemos en el try-catch) será recuperar la clase del evento correspondiente al nombre con el que se guardó en BD (Ojo 👀! Estamos conformes en que en lugar de una RuntimeException deberíamos utilizar en este caso una excepción particular para no silenciar así otro tipo de errores que pudieran producirse). Una vez que recuperamos la clase podremos utilizar el método estático _fromPrimitives()_ para instanciar el evento, recordemos que la [implementación de este método](https://github.com/CodelyTV/php-ddd-skeleton/blob/master/src/Mooc/Courses/Domain/CourseCreatedDomainEvent.php#L40-L47) será particular para cada evento de dominio.

Finalmente, con el evento ya montado se lo pasaremos al _subscribers()_ para que ejecute todos los eventos que haya asociados.

¿Alguna Duda?
=============

Si tienes alguna duda sobre este video o quieres compartir alguna sugerencia no dudes en abrir una nueva discusión más abajo 👇👇👇

¡Nos vemos en el siguiente video: 🤖 Comando para el consumidor de eventos de dominio!