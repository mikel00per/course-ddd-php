Inyección de dependencias: Controller como servicio
===================================================

Vamos a utilizar la clase `RandomNumberGenerator` como ejemplo de lo que podría ser la necesidad de interactuar con un colaborador desde el Controller a fin de evitar meter en éste toda la lógica que necesitemos, y lo haremos siguiendo el principio de _Composition over Inheritance_ que ya vimos en SOLID

Clase `HealthCheckGetController`:

    final class HealthCheckGetController
    {
        private $generator;
    
        public function __construct(RandomNumberGenerator $generator)
        {
            $this->generator = $generator;
        }
        public function __invoke(Request $request): Response
        {
            return new JsonResponse(
                [
                    'mooc-backend' => 'ok',
                    'rand'         => $this->generator->generate(),
                ]
            );
        }
    }


Lo que haremos será definir en el constructor que vamos a recibir el la clase colaboradora como parámetro en el constructor de nuestro Controller (De momento será simplemente una instancia de esa clase) y utilizarla en el `__invoke`. Esta inyección mágica se produce gracias al **Autowiring** con el que cuentan muchos frameworks

Fichero `services.yaml`:

    services:
      _defaults:
        autoconfigure: true
        autowire: true
    
    
      CodelyTv\Mooc\Backend\Controller\:
        resource: '../src/Controller'
        tags: ['controller.service_arguments']
    
      CodelyTv\:
        resource: '../../../../src'


El punto donde se establece esta magia es el [fichero services](https://github.com/CodelyTV/php-ddd-skeleton/blob/0.3.0/applications/mooc/backend/config/services.yaml) dentro de la carpeta de configuración. Al igual que sucedía con las rutas, podremos definir que este fichero yaml incluya otros múltiples ficheros por cada módulo dentro de una carpeta services

El atributo ‘autowire’ establecido a true lo que permite es que si detecta una dependencia, la inyectará. En el caso de encontrar una Interface, inyectará automáticamente la implementación si sólo tuviera una, en el caso de que hubiera más de una implementación, tendremos que configurar cual queremos que inyecte

Para que esto sea posible, debemos indicarle que cargue el directorio ‘src’ con el namespace ‘CodelyTv’, lo cual hará que se creen servicios con este namespace (Gracias al paso dado por Symfony en la versión 3.2)

Un aspecto importante en esta generación de servicios, es que Symfony en tiempo de compilación elimina aquellos servicios que no se inyectan en ninguna parte (Por ejemplo `UserId`) gracias al **inline** 🗑. Para este proyecto no vemos necesario definir ninguna regla de exclusión adicional, sino que será suficiente con la función realizada por el inline

Siempre que creemos nuevos ficheros de servicios por alguna necesidad como añadir configuraciones específicas para los módulos tendremos que volver a definir en dichos ficheros la clave `_defaults` al principio con el `autoconfigure`y `autowire` de nuevo a true, ya que el scope que tiene esta clave es a nivel de fichero

Por otra parte, queremos definir nuestro Controller como un servicio y que no tenga acoplamiento con el framework, para ello lo que haremos será tagearlo como controlador. El `autoconfigure` nos permite que, en lugar de añadir el tag `controller.service_arguments` en cada controlador, lo añada automáticamente a todo lo que haya dentro de la ruta que le especificamos, en este caso _‘src/Controller’_. La otra opción habría sido extender nuestro Controller de una clase Symfony Controller, pero implica un acoplamiento que no queremos tener en este caso

Toda esta configuración viene cargada por el Kernel MoocBackendKernel, dónde especifiquemos los siguientes aspectos:

*   Cargar todos los [ficheros de configuración](https://github.com/CodelyTV/php-ddd-skeleton/blob/0.3.0/applications/mooc/backend/src/MoocBackendKernel.php#L41) con extensión xml o yaml que existan en la carpeta _‘/config’_
*   A la hora de transpilar todos los servicios a ficheros PHP [realizará el _inline_](https://github.com/CodelyTV/php-ddd-skeleton/blob/0.3.0/applications/mooc/backend/src/MoocBackendKernel.php#L38) que comentábamos previamente
*   Sobreescribir el método configureRoutes para [indicar dónde tendremos las rutas](https://github.com/CodelyTV/php-ddd-skeleton/blob/0.3.0/applications/mooc/backend/src/MoocBackendKernel.php#L46-L48)

¿Alguna Duda?
=============

Si tienes alguna duda sobre el contenido del video o quieres compartir alguna sugerencia no dudes en abrir una nueva discusión más abajo 👇👇👇

¡Nos vemos en el siguiente video! ✅ Añadir test de aceptación con Behat