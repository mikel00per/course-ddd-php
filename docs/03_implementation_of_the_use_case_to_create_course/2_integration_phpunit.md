Integrar PHPUnit para tests unitarios
=====================================

Antes de adentrarnos más en la implementación de nuestro caso de uso, veremos cómo integrar PHPUnit en un proyecto para realizar tests unitarios.

En primer lugar necesitamos requerir la dependencia, lo haremos desde la consola con el siguiente comando para asegurarnos de que agrega en nuestro fichero `composer.json` la última versión y que además permite su actualización sin romper compatibilidades

    composer require --dev phpunit/phpunit


Además creamos en la raiz el fichero `phpunit.xml` para especificar las reglas que queremos que siga al ejecutar los tests unitarios. Es interesante sobrescribir aquellas propiedades que, aunque vengan con el valor que buscamos por defecto, queremos asegurarnos de que van a mantener ese valor por si en algún momento esa configuración por defecto de phpunit cambia silenciosamente.

Dentro de este mismo fichero indicaremos también dentro de la etiqueta `testsuites` el directorio donde encontrará nuestros tests, aquí podríamos crear una suite para cada Bounded Context o incluso por cada tipologia de test pero consideramos más cómodo englobarlos en una sóla mientras tengamos poco código. Separar distintas testsuites y por las diferentes tipologías de tests nos abrirá la posibilidad de lanzar tests en paralelo para así conseguir un proceso mucho más rápido

Una vez tenemos lista la configuración podremos crear nuestros ricos tests unitarios dentro del directorio _tests/src/_ tal como le habíamos indicado previaamte en el fichero xml. Recordad que los tests deben extender de `TestCase` si queremos que PHPUnit los reconozca como tal

*   **CodelyTV Tips** ☝️:
    *   Para evitar tener que ejecutar manualmente todos los comandos necesarios para paralelizar los tests podríamos definirlos en el fichero Makefile bajo un sólo comando del tipo ‘make test’
    *   Si queremos utilizar snakeCase en el nombre de los métodos de nuestros tests, es necesario que añadamos la anotación @test

¿Alguna Duda?
=============

Si tienes alguna duda sobre el contenido del video o quieres compartir alguna sugerencia no dudes en abrir una nueva discusión más abajo 👇👇👇

¡Nos vemos en el siguiente video: Implementación del caso de uso y test unitario!