Añadir Tests de Aceptación con Behat
====================================

Metiéndonos en Contexto 🤔
--------------------------

Cursos relacionados y recomendados para entender mejor qué tipología de test utilizaremos en las diferentes capas de nuestra aplicación y por qué hacerlo de esta manera

*   [🎯 Arquitectura Hexagonal](https://pro.codely.tv/library/arquitectura-hexagonal/66748/about/)
*   [✅ Testing: Introducción y buenas prácticas](https://pro.codely.tv/library/testing-introduccion-y-buenas-practicas/90916/about/)

![sistema_colas_mensajeria](https://cdn.filestackcontent.com/plr2nCBS9mQu4tpyUvvA)

Los distintos tipos de tests de los que disponemos nos van a permitir cubrir diferentes aspectos del flujo de petición dentro de nuestra aplicación:

*   Test Unitario: Atacará desde el Caso de Uso hasta las Interfaces (que actúan a modo de barrera del Dominio)
*   Test de Integración: Su objetivo será la implementación concreta de una Interface
*   Test de Aceptación: Probará la aplicación completa desde el Controller hasta la implementación de las interfaces, tratando de ser lo más realista posible

Añadiendo Behat ✅
-----------------

Podéis consultar como queda la aplicación con Behat en [este tag](https://github.com/CodelyTV/php-ddd-skeleton/tree/0.5.0) del repo que estamos siguiendo para el curso

Test `health_check_get.feature`:

    Feature: Api status
      In order to know the server is up and running
      As a health check
      I want to check the api status
    
      Scenario: Check the api status
        Given I send a GET request to "/health-check"
        Then the response content should be:
        """
        {
          "mooc-backend": "ok",
          "rand": 1
        }
        """


Dentro de la Feature podemos encontrar múltiples escenarios, que serán los diferentes tests que queremos pasar. Para cada escenario se darán las **Given-When-Then** que en el ejemplo podemos interpretar como “Dada esta situación, Cuando suceda una petición GET a nuestro punto de entrada, Entonces esperamos que el resultado sea esta respuesta en formato Json”

Para hacer la magia necesaria que interprete esta sintaxis como determinadas acciones (Por ejemplo realizar una petición http GET) utilizamos la clase `ApiRequestContext`, que ya tenéis creada en el repo, además de un conjunto de [dependencias](https://github.com/CodelyTV/php-ddd-skeleton/blob/0.5.0/composer.json#L25-L28)

*   _behat & mink-extension_: Son las dependencias básicas que necesitaremos para utilizar Behat en nuestro proyecto
*   _mink-browserkit-driver_: Nos ahorrará tener que levantar un servidor local con la aplicación al que realizar las peticiones y un thread paralelo en el que se ejecuten los tests

Para que Behat entienda estos ficheros feature y los traduzca en métodos PHP, al igual que nos sucedía con la estructura de carpetas de Symfony, haremos uso de una configuración personalizada que nos permita mimificar la estructura de _app/_. En concreto necesitamos crear el fichero behat.yml en la raiz del proyecto e importar por cada aplicación su fichero yaml de configuración de los tests

Fichero `mooc_backend.yml`:

    mooc_backend:
      extensions:
        FriendsOfBehat\SymfonyExtension:
          kernel:
            class: CodelyTv\Apps\Mooc\Backend\MoocBackendKernel
          bootstrap: apps/mooc/bootstrap.php
        Behat\MinkExtension:
          sessions:
            symfony:
              symfony: ~
          base_url: ''
    
      suites:
        health_check:
          paths: [ tests/apps/mooc/backend/features/health_check ]
          contexts:
            - CodelyTv\Tests\Shared\Infrastructure\Behat\ApiRequestContext
            - CodelyTv\Tests\Shared\Infrastructure\Behat\ApiResponseContext


Dentro de este fichero lo primero que haremos será añadir la extensión de Symfony para mejorar la integración con el framework, indicando cual es el Kernel del que debe tirar y el bootstrap a utilizar (se encargará tanto del [autoload](https://getcomposer.org/doc/04-schema.md#autoload) como de cargar las variables de entorno). Además cargaremos también la extensión de Mink para la simulación de las requests Finalmente añadimos nuestras suites de tests, indicando las rutas donde encontrarlos y las dependencias que puedan requerir dichos tests. Si examinamos estas dependencias, comprobaremos que es aquí donde estamos conectando con la clase `ApiRequestContext` mencionábamos antes

Clase `ApiRequestContext`:

    final class ApiRequestContext extends RawMinkContext
    {
        private $request;
        public function __construct(Session $session)
        {
            $this->request = new MinkSessionRequestHelper(new MinkHelper($session));
        }
        /**
         * @Given I send a :method request to :url
         */
        public function iSendARequestTo($method, $url): void
        {
            $this->request->sendRequest($method, $this->locatePath($url));
        }
        /**
         * @Given I send a :method request to :url with body:
         */
        public function iSendARequestToWithBody($method, $url, PyStringNode $body): void
        {
            $this->request->sendRequestWithPyStringNode($method, $this->locatePath($url), $body);
        }


Sobre cada función de esta clase se define la expresión que usaremos desde el test, indicando las variables con el prefijo ‘:’ que se extrapolarán a parámetros de la función

Sobrecargando Implementaciones para los Test de Aceptación 🤹
-------------------------------------------------------------

Al igual que sucede en el caso de nuestro `RandomNumberGenerator`, aunque queremos que nuestras pruebas sean lo más realistas posibles, habrá situaciones en las que no podremos controlar la respuesta concreta que recibiremos. En estos casos, lo que haremos será sobrecargar la implementación dentro del namespace de test para que nos devuelva un valor constante.

Fichero `services_test.yaml`:

    framework:
      test: true
    
    services:
      _defaults:
        autoconfigure: true
        autowire: true
    
      CodelyTv\Tests\:
        resource: '../../../../tests/src'
    
      CodelyTv\Shared\Domain\RandomNumberGenerator: '@CodelyTv\Tests\Shared\Infrastructure\ConstantRandomNumberGenerator'


Para que esto funcione, tendremos que indicar en nuestra definición de servicios de test que cuando detecte la interface `RandomNumberGenerator` lo que debe inyectar es nuestra clase `ConstantRandomNumberGenerator` (realmente estamos sobreescribiendo la definición de esta implementación). Tal como hicimos con el fichero `services` en el video anterior, será necesario cargar este nuevo `services_test` en el Kernel de la aplicación

Otro caso en el que puede ser necesario este ‘falseo’ de las implementaciones es cuando realizamos despliegues progresivos de nuevas features y queremos ir testeandolo, de modo que determinemos qué es lo que debe probarse (dirigiremos el test a adoptar una implementación ‘nueva’ cuando queramos testear la nueva feature y tomar la implementación ‘antigua’ cuando lo que testemos sea el código antiguo)

¿Alguna Duda?
=============

Si tienes alguna duda sobre el contenido del video o quieres compartir alguna sugerencia no dudes en abrir una nueva discusión más abajo 👇👇👇

¡Nos vemos en la siguiente lección! ♻️ Desarrollo Outside-in: Implementación del caso de uso para crear curso