🎍 Integración con Twig
=======================

Configurando Twig 🎍
--------------------

Twig es el sistema de plantillas que utilizaremos para trabajar en el frontend de nuestro backoffice, se trata del estándar utilizado por Symfony a la hora de renderizar sitios web desde el lado del servidor (Podeis introduciros en otros frameworks del lado front con los cursos de [Vue](https://pro.codely.tv/library/crea-una-app-con-vuejs-y-jest-aplicando-tdd/65206/about/) y [React](https://pro.codely.tv/library/reactjs-de-0-a-deploy-siguiendo-buenas-practicas/78993/about/))

Para su instalación seguimos el mismo proceso que hemos venido llevando a cabo con el resto de dependencias

    composer require symfony/twig-bundle --ignore-platform-reqs


*   **CodelyTV Tip** ☝️: Si modificamos el orden de las dependencias en el composer.json se produce un conflicto con el composer-lock.json al cambiar el hash del fichero, por lo que debemos actualizar éste también ejecutando el comando (`composer update --lock`)

Fichero \`bundles.php\`\`

    <?php
    $bundles = [
        Symfony\Bundle\FrameworkBundle\FrameworkBundle::class                              => ['all' => true],
        FriendsOfBehat\SymfonyExtension\Bundle\FriendsOfBehatSymfonyExtensionBundle::class => ['test' => true],
        Symfony\Bundle\TwigBundle\TwigBundle::class                                        => ['all' => true],
    ];
    
    $suggestedBundles = [];
    if (true) {
        $suggestedBundles[WouterJ\EloquentBundle\WouterJEloquentBundle::class] = ['test' => true];
    }
    
    return array_merge($bundles, $suggestedBundles);


Una vez instalado el bundle de twig, es necesario que demos de alta en el fichero de bundles, especificando en el array asociativo de Bundle-Entorno que lo activaremos en todos los entornos

Por último también tendremos que definirle a twig dónde encontrar su configuración dentro del \`services.yaml\`\`

    twig:
      default_path: '%kernel.project_dir%/templates'
      strict_variables: true


Con estos dos atributos le estaremos indicando por un lado la ruta donde encontrar las plantillas (default\_path) y, por otro lado, que sea estricto a la hora de requerir variables en las plantillas (strict\_variables), lo que se traducirá en que lanzará un error si una plantilla no recibe alguna variable que estuviera esperando

Renderizando nuestra Home! 🏠
-----------------------------

Una vez configurado el sistema de plantillas podemos pasar a probarlo creando una vista muy simple para la Home de nuestro backoffice

En primer lugar crearemos un `HomeGetController` siguiendo la idea de incluir la nomenclatura acoplada al tipo de comunicación con el usuario. Este controlador recibirá por constructor el Enviroment de twig (la llamaremos simplemente `twig` para evitar confusiones con las propias variables de entorno)

Desde el _\_\_invoke()_ devolveremos en la respuesta la llamada al _render()_ de twig que recibe por parámetros la ruta relativa de la plantilla y un array con las variables que dicha plantilla pueda necesitar

Dentro del directorio de rutas tendremos que definir la ruta de la home para llamar a este controlador y que nos devuelva la vista especificada previamente

¿Alguna Duda?
=============

Si tienes alguna duda sobre este video o quieres compartir alguna sugerencia no dudes en abrir una nueva discusión más abajo 👇👇👇

¡Nos vemos en el siguiente video: ⑩ Contador de cursos vía query desde Controller!