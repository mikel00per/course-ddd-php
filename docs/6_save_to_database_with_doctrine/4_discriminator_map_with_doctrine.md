Discriminator Map con Doctrine
==============================

Uno de los últimos temas que veíamos en el curso de [Principios SOLID Aplicados](https://pro.codely.tv/library/principios-solid-aplicados/77070/about/) era el _patrón ‘Specification’_ o _patrón ‘Criteria’_, y esto es algo en lo que Doctrine nos va a permitir trabajar y que veremos cómo aplicarlo en este video con el **Discriminator Map**

*   Podéis encontrar el código relativo a este video en el repositorio de [php-dd-example](https://github.com/CodelyTV/php-ddd-example)

Modelando nuestros Steps 👣
---------------------------

Jerarquía derivada de Step:

    Domain
      |-Challenge
      |    |-ChallengeStep.php
      |    |-ChallengeStepStatement.php
      |-Quiz
      |    |-QuizStep.php
      |    |-QuizStepAnswer.php
      |    |-QuizStepQquestion.php
      |-Video
      |    |-VideoStep.php
      |    |-VideoStepText.php
      |-Step
      |-StepEstimatedDuration.php
      |-StepId.php
      |-StepOrder.php
      |-StepPoints.php
      |-...


A nivel de modelado de nuestro dominio vemos cómo el agregado `Step` y sus correspondientes Value Objects están definidos en la raíz de la carpeta Domain, y que por cada tipo específico hemos creado una carpeta

A nivel de Infraestructura, lo que tendremos en [nuestra BD](https://github.com/CodelyTV/php-ddd-example/blob/0d1f9f43536ec98c6a285a7c9a08974dd12a16a6/databases/mooc.sql#L17) será una tabla ‘base’ con todos los atributos comunes, en esta tabla definiremos también en la colmna `type` el tipo de Step del que se trata para hacer luego el mapeo con Doctrine. Por otro lado, tendremos una tabla con cada uno de los diferentes tipos de Step y sus particularidades. Seguir este tipo de Mapeo tiene la ventaja de seguir un diseño muy similar al que tenemos a nivel de clases. Por contra, será necesario realizar JOINs con estas tablas cuando tengamos que recuperarlos

Otras alternativas podrían ser guardar todos los campos en una única tabla (lo cual nos ahorraría los JOINs pero implicaría permitir que hubiera muchos campos nullables en nuestra tabla), guardar la información dentro de un Json en una columna y transformarla una vez la recuperamos, o utilizar directamente distintas tablas por cada tipo de Step sin que exista una tabla ‘base’ (lo que nos obligaría a modificar todas esas tablas cada vez que necesitemos modificar uno de los atributos comunes)

Mapeando los distintos Steps 🗺
-------------------------------

    CodelyTv\Mooc\Steps\Domain\Step:
      type: entity
      inheritanceType: JOINED
      discriminatorColumn:
        name: type
        type: smallint
      discriminatorMap:
        1: CodelyTv\Mooc\Steps\Domain\Challenge\ChallengeStep
        2: CodelyTv\Mooc\Steps\Domain\Quiz\QuizStep
        3: CodelyTv\Mooc\Steps\Domain\Video\VideoStep
      table: steps
    
      // ....


En el fichero [yaml de configuración](https://github.com/CodelyTV/php-ddd-example/blob/master/src/Mooc/Steps/Infrastructure/Persistence/Step.orm.yml) de Step lo que haremos será indicarle que se trata de una entidad de tipo JOINED y que utilizará como columna discriminadora a `type`, tal como veíamos en BD, de modo que le estaremos indicando a través de esta columna con qué clase debe hacer el Join. Además de esto, especificaremos en este fichero el resto de atributos comunes de todos los steps. Por otro lado, definieremos otro yaml por cada embeddable y los distintos tipos de Steps, utilizando como prefijo el nombre de la clase (Recordad que habíamos creado una carpeta en Domain por cada tipo de Step)

Clase `StepRepository`:

    interface StepRepository
    {
        public function save(Step $step): void;
    }


Finalmente, para nuestro repositorio vemos que toda esta configuración es completamente transparente y de lo único que se preocupará es de almacenar un Step o cualquier clase que extienda de Step

¿Alguna Duda?
=============

¡Ya hemos cerrado la integración con Doctrine! Así que ahora empezaremos a ver qué sucede con DDD y Active Record a través de Eloquent y seguiremos iterando para añadir eventos de dominio síncronos y asíncronos, añadiremos RabbitMQ, gestión de errores y mucho más en las siguientes lecciones

Si tienes alguna duda sobre este video o quieres compartir alguna sugerencia no dudes en abrir una nueva discusión más abajo 👇👇👇

¡Nos vemos en la siguiente lección: 😅 Alternativa: Repositorios con Active Record - Eloquent!