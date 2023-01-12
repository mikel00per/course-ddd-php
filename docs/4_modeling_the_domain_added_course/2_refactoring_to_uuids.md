Refactoring a UUIDs como identificadores
========================================

Seguimos con el modelado de nuestro dominio, centrándonos ahora en refactorizar los identificadores de nuestros recursos al estándar de los UUIDs

*   Podéis encontrar el código relativo a este video en el tag [0.9.0 del repo](https://github.com/CodelyTV/php-ddd-skeleton/tree/0.9.0)

Para esta refactorización utilizaremos la librería más estandarizada en PHP, _ramsey/uuid_, requiríendola a través de la consola tal como ya hemos visto en videos anteriores

    composer require ramsey/uuid


Dentro del agregado `Course` sustituiremos el String que recibíamos como identificador por un nuevo Value Object `CourseId` que simplemente heredará de la clase `Uuid`. Este Value Object es un gran candidato para estar dentro del Share Kernel debido a sus implicaciones, pero de momento lo mantendremos dentro del Dominio del Bounded Context de _Mooc_

Clase `Uuid`:

    use InvalidArgumentException;
    use Ramsey\Uuid\Uuid as RamseyUuid;
    class Uuid
    {
        private $value;
        public function __construct(string $value)
        {
            $this->ensureIsValidUuid($value);
            $this->value = $value;
        }
        public static function random(): self
        {
            return new self(RamseyUuid::uuid4()->toString());
        }
        public function value(): string
        {
            return $this->value;
        }
        private function ensureIsValidUuid($id): void
        {
            if (!RamseyUuid::isValid($id)) {
                throw new InvalidArgumentException(sprintf('<%s> does not allow the value <%s>.', static::class, $id));
            }
        }
        public function __toString()
        {
            return $this->value();
        }
    }


Esta clase actúa como wrapper del vendor externo que comentábamos al principio. Entonces ¿Estamos violando la regla de independencia? ¿Por qué permitimos este acoplamiento? 🤯 ¡Tranquilos! Hay una explicación 🙌 Dentro de Uuid tenemos una lógica de validación que ya nos provee esta librería, la cual nos parece más interesante que replicarla y mantenerla dentro de nuestro código. La manera en que contenemos esta ‘contaminación’ de una librería externa es encapsularla dentro de esta clase `Uuid`

La **diferencia con los repositorios**, en los que si haríamos una inversión de dependencias, es que en el caso del Uuid forma parte del modelo de dominio y tendríamos que **inyectar esta dependencia** en el constructor, por lo que cada vez que recuperásemos un listado de Cursos del repositorio, estaríamos haciendo múltiples inyecciones, lo cual sería una locura 🤷🏼‍♂️. Otro tip que puede ayudar a ver si estamos patinando demasiado es pensar si la clase donde inyectamos la librería **toca Input/Output**

En otros lenguajes como Java contamos con una clase de tipo Uuid dentro del propio sistema, por lo cual podríamos utilizarla sin preocuparnos en este tipo de dilemas, pero al no ser el caso en PHP, debemos decidir si recurrir a este tipo de librerías externas o añadir ese código dentro de nuestra aplicación

Finalmente utilizaremos `CourseId` en todos los sitios de nuestra aplicación en los que estuvieramos haciendo un `new Course` en lugar del String que previamente enviábamos, allí donde utilizaramos el identificador ahora lo recogeremos a través de la función _value_ del propio Value Object

Con este refactor estamos ganando en semántica y robustez (aportando las reglas de integridad que nos da el Uuid). Veremos cómo hacer lo mismo con los demás atributos y cómo reducir el impacto que estos tienen sobre los tests unitarios

¿Alguna Duda?
=============

Si tienes alguna duda sobre el contenido del video o quieres compartir alguna sugerencia no dudes en abrir una nueva discusión más abajo 👇👇👇

¡Nos vemos en el siguiente video! 👉 Value Objects: Inmutabilidad y tips para agilizar desarrollo