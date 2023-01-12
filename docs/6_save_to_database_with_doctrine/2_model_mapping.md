Mapeo de modelos en Doctrine Custom Types vs Embeddables
========================================================

El siguiente paso en nuestra integración con Doctrine será ver que son los **Custom Types** y **Embeddables**, conociendo además cual de ellos utilizar en base a lo que vayamos a modelar. Será fácil identificar con cual de ellos nos manejaremos siguiendo este árbol de decisiones

*   Podéis encontrar el código relativo a este video en el tag [0.13.0 del repo](https://github.com/CodelyTV/php-ddd-skeleton/tree/0.13.0)

### ¿Cuándo usar cada uno? 🤔

![decision-tree](https://cdn.filestackcontent.com/lW1lpKYZRGogHGRV1NOA)

*   **Custom Type**
    *   ID: Puesto que Doctrine nos obliga a definir el tipo para los ids, la decisión es sencilla
    *   Parámetros nullables: Hasta la fecha, Doctrine no nos permite crear Embeddables nullables
    *   Elementos de serialización-deserialización compleja (por ejemplo eventos de dominio)
*   **Embedable**: Parámetros no nullables

### Conociendo los Custom Types y Embeddables 🤝

El momento en que esta diferenciación entra en juego es justo en el proceso de conversión de los elementos desde PHP a la BD y viceversa

En el caso de los Embeddables como [CourseName](https://github.com/CodelyTV/php-ddd-skeleton/blob/0.13.0/src/Mooc/Courses/Infrastructure/Persistence/Mappings/CourseName.orm.xml), estaríamos hablando de ‘incrustar’ una tabla dentro de un único campo utilizando para ello un fichero de configuración (estos Embeddables se reflejarán dentro del mapeo superior de la entidad)

Por otra parte, en el caso de los Custom Types, lo que estaríamos haciendo es valernos de los Types de los que ya nos provee Doctrine, que cuenta con los métodos para transformar los valores de PHP a BD y viceversa

Clase `CourseIdType`:

    final class CourseIdType extends UuidType
    {
        public static function customTypeName(): string
        {
            return 'course_id';
        }
        protected function typeClassName(): string
        {
            return CourseId::class;
        }
    }


En el caso concreto de los Uuid, hemos creado una clase abstracta `UuidType` que además de heredar de `StringType` nos obliga a implementar los métodos _customTypeName_ y _typeClassName_ que nos servirán a la hora de mapear la entidad mayor

Mapeo `Course`:

    <doctrine-mapping xmlns="http://doctrine-project.org/schemas/orm/doctrine-mapping"
                      xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
                      xsi:schemaLocation="http://doctrine-project.org/schemas/orm/doctrine-mapping
                              https://www.doctrine-project.org/schemas/orm/doctrine-mapping.xsd">
    
        <entity name="CodelyTv\Mooc\Courses\Domain\Course" table="courses">
            <id name="id" type="course_id" column="id" length="36" />
    
            <embedded name="name" class="CodelyTv\Mooc\Courses\Domain\CourseName" use-column-prefix="false" />
            <embedded name="duration" class="CodelyTv\Mooc\Courses\Domain\CourseDuration" use-column-prefix="false" />
        </entity>
    
    </doctrine-mapping>


A la hora de mapear el agregado Curso indicamos tanto los custom types como los embeddables que lo compongan. Como por defecto Doctrine añade a cada embeddable un prefijo con el nombre de la entidad, debemos añadir el atributo _use-column-prefix_ a false

# ¿Alguna Duda?

Si tienes alguna duda sobre este video o quieres compartir alguna sugerencia no dudes en abrir una nueva discusión más abajo 👇👇👇

¡Nos vemos en el siguiente video: Implementación del repositorio para MySQL y test de integración!