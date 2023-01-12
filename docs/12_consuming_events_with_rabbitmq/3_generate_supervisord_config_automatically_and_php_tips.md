🤓 Generar config SupervisorD automáticamente y tips PHP
========================================================

Creando un comando para consumir los eventos 🚌
-----------------------------------------------

Al igual que vimos en el caso de MySQL el modo en que llamaremos a nuestro consumidor de RabbitMQ es mediante un [comando de consola](https://github.com/CodelyTV/php-ddd-skeleton/blob/master/apps/mooc/backend/src/Command/DomainEvents/RabbitMq/ConsumeRabbitMqDomainEventsCommand.php)

Este comando recibirá por constructor, además del consumer, el `DatabaseConnections` con el que trabajaremos (y limpiaremos para evitar condiciones de carrera) y el `DomainEventSubscriberLocator` que nos va a facilitar la asociación entre las distintas colas y casos de uso

A la hora de **configurar** y ejecutar este comando lo que vamos a necesitar recibir será tanto el **nombre de la cola** como la **cantidad de eventos** que queremos consumir (recordad que esta cantidad está ligada a optimizar el consumo de la memoria). Si en alguna de las veces en que se ejecute el proceso no encontrase eventos que consumir, se mataría el proceso para volver a levantarse a la espera de nuevo de eventos listos

Por cada una de las veces que le hemos indicado ejecutaremos el método _consumer()_ que lo que hará en primer lugar será recuperar el subscriber a partir del nombre de la cola que habíamos recibido (Podéis verlo en [esta](https://github.com/CodelyTV/php-ddd-skeleton/blob/55a7764e2feb26e1966b63f8ffd8e3b9e3f6428f/src/Shared/Infrastructure/Bus/Event/DomainEventSubscriberLocator.php#L30) función). Una vez lo tenemos llamaremos al _consume()_ de nuestro `RabbitMqDomainEventsConsumer` pasándole tanto el subscriber recuperado como el nombre de la cola. Si ese proceso se lleva a cabo sin errores limpiaremos la conexión y volveremos a ejecutar la función la cantidad de veces que nos reste

Añadiendo Supervisor para automatizar tareas 🕵
-----------------------------------------------

Vamos a utilizar **Supervisor** para automatizar y gestionar mejor este proceso. Para crear el tipo de fichero que necesita para cargar Supervisor hemos creado [este](https://github.com/CodelyTV/php-ddd-skeleton/blob/master/apps/mooc/backend/src/Command/DomainEvents/RabbitMq/GenerateSupervisorRabbitMqConsumerFilesCommand.php) comando. Al ejecutarlo nos va a generar un fichero con el nombre del caso de uso

Ejemplo fichero supervisor:

    [program:codelytv_codelytv.mooc.courses_counter.increment_courses_counter_on_course_created]
    command         = /var/www/apps/mooc/backend/bin/console codelytv:domain-events:rabbitmq:consume --env=prod codelytv.mooc.courses_counter.increment_courses_on_course_created 200
    process_name    = %(program_name)_$(process_num)02d
    numprocs        = 1
    startsecs       = 1
    startretries    = 10
    exitcodes       = 2
    stopwaitsecs    = 300
    autostart       = true


Esto nos va a permitir que una vez consumidos 200 eventos Supervisor mate el proceso y vuelva a levantarlo. Además estamos especificando que sólo levante un proceso en paralelo, si el proceso se levanta y muere en menos de 1 segundo (startsecs) lo reintentará 10 veces antes, si en estos reintentos sigue muriendo en menos de 1 segundo matará el proceso con un exitcode ‘2’ y esperará 300s para volver a levantar el proceso

Si volvemos al comando veremos que lo que hace una vez ejecutado es crear un fichero de configuración por cada subscriber recuperado, reemplazando los valores dentro del template que tenemos preparado dentro del mismo comando, con lo cual será infinítamente sencillo crear todos los ficheros necesarios y dejárselos preparados a Supervisor para que se ocupe de forma automática 👌

¿Alguna Duda?
=============

Si tienes alguna duda sobre este video o quieres compartir alguna sugerencia no dudes en abrir una nueva discusión más abajo 👇👇👇

¡Nos vemos en el siguiente video: 🙅🏾‍♂️ Gestión de errores al consumir con RabbitMQ: Colas de retry y dead letter!