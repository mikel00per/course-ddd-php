Value Objects: Inmutabilidad y tips para agilizar el desarrollo
===============================================================

Ya vimos en el [curso de DDD Aplicado](https://pro.codely.tv/library/domain-driven-design-ddd//about/) las ventajas que nos ofrecía el uso de los Value Objects, así que ahora nos centraremos en cómo implementarlos en nuestra aplicación y cómo agilizar el desarrollo con estos elementos

*   Podéis encontrar el código relativo a este video en el tag [0.9.0 del repo](https://github.com/CodelyTV/php-ddd-skeleton/tree/0.9.0)

Tal como vimos en el caso de CourseId en el video anterior, refactorizar el resto de atributos de Curso a Value Objects se traducirá en mayor semántica y robustez, ya que de este modo en la propia firma del constructor de la clase ya nos queda claro que tendremos que pasarle un nombre y una duración válidos. Además, esto nos dará la confianza necesaria para ahorrarnos ese tipo de validaciones dentro de la clase Curso

Clase `CourseName`:

    use CodelyTv\Shared\Domain\ValueObject\StringValueObject;
    
    final class CourseName extends StringValueObject
    {
    
    }


Para facilitar el desarrollo de Value Objects que sólo tendrán un valor de tipo String hemos creado una clase base `StringValueObject` (haríamos algo similar en caso de contener simplemente un Integer)

Clase `StringValueObject`:

    abstract class StringValueObject
    {
        protected $value;
        public function __construct(string $value)
        {
            $this->value = $value;
        }
        public function value(): string
        {
            return $this->value;
        }
        public function __toString()
        {
            return $this->value();
        }
    }


Esta clase base se asegura de que el parámetro que recibirá es de tipo String además de proporcionar un método getter (por convenio definimos como _value_ el campo en los casos en que se envuelva un único valor) y un _toString_

Además de heredar de esta clase base, nuestro CourseName podría realizar otras validaciones como el tamaño mínimo/máximo del nombre, para lo cual sobrescribiríamos el constructor para añadir las clausulas de guarda necesarias

    final class CourseName extends StringValueObject
    {
        public function __construct(string $value)
        {
            $this->ensureLengthIsInferiorThan30Characters($value);
    
            parent::__construct($value);
        }
    
        private function ensureLengthIsInferiorThan30Characters(string $value): void
        {
            if(strlen($value) > 30) {
                throw new \InvalidArgumentException('The CourseName <%s> has more than 30 Characters', $value)
            }
        }
    }


La clausula simplemente lanzará una Exception en caso de no pasar la comprobación. El hecho de definirla en positivo nos facilitará su legibilidad dentro del código. Otro aspecto de interés será utilizar tipos especíicos de Exceptions, que nos permitirán darles un tratamiento distinto en función de la que capturemos

Una vez creados todos nuestros Value Objects, tendremos que actualizar tanto el servicio de aplicación como los tests. En el caso del Controlador no habrá ningún cambio, ya que las modificaciones en el DTO pertenecen al dominio y no queremos que en ningún caso una capa tan alejada conozca sobre éste

¿Alguna Duda?
=============

Si tienes alguna duda sobre esta lección o quieres compartir alguna sugerencia no dudes en abrir una nueva discusión más abajo 👇👇👇

¡Nos vemos en la siguiente lección: ✅ Modelando el dominio: Implicaciones en tests!