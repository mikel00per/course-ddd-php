📁 Estructura de carpetas y acelerando la creación del proyecto 🚀
==================================================================

En este video os proponemos una manera de estructurar vuestros proyectos orientados a **DDD** y **Arquitectura Hexagonal** además de traer una herramienta Open Source que os ayudará en este proceso. Para lanzar la herramienta es tan sencillo como usar el comando que tenéis aquí abajo 👇

    composer create-project codelytv/ddd-skeleton:0.4.0


Esto descargará todas las dependencias necesarias y montará el esqueleto para que empieces a programar desde estas clases base 👨🏽‍💻(si no le especificamos ningún nombre en el comando, la carpeta del proyecto se creará con el nombre ddd-skeleton)

Si revisamos el arbol de directorios podremos intuir rápidamente que está orientado a trabajar con un monorepositorio en el que recogemos todas las apps (inicialmente generará la estructura de carpetas para _mooc_ y _backoffice_, que podremos cambiar en base a nuestras necesidades)

Dominio de la aplicación
------------------------

![contexto](https://cdn.filestackcontent.com/oUiqymLS42M3oxOfvnM8)

En nuestra aplicación identificamos tres **Bounded Contexts** (profundizamos mucho más sobre estos conceptos en el curso de [Domain-Driven Design Aplicado](https://pro.codely.tv/library/domain-driven-design-ddd/87157/about/)), estos derivan de la estructura organizacional o los equipos de nuestra empresa, es decir, mapeamos desde la aplicación estos conceptos del mundo real

Dentro de estos Bounded Context encontraremos diferentes **Módulos** que en ocasiones coincidirán con los módulos de otro Bounded Context

También encontraremos las **Aplicaciones**, que se sitúan fuera de los Bounded Contexts. Estas aplicaciones serán los puntos de entrada (Controladores) que reciban las peticiones de usuarios finales y que interactuarán con los diferentes Contextos

Nosotros proponemos estructurar todo dentro de un monorepo pues estando en un mismo lenguaje resultará mas fácil de gestionar, pero también podría organizarse en múltiples repos si por ejemplo quisieramos programar cada Bounded Context en un lenguaje distinto

Estructura de Carpetas
----------------------

![directorioRaiz](directorioRaiz.png "directorioRaiz")

Volviendo al árbol de directorios del proyecto encontramos en la raiz tres carpetas:

*   **apps**: Contiene las Aplicaciones de cada contexto
*   **src**: Contiene los distintos Bounded Contexts
*   **tests**: Contiene una mimificación de los directorios de ese mismo nivel (es decir, apps y src)

Para cada aplicación podemos ver la misma división de carpetas para el backend y el frontend, y por supuesto en la carpeta tests encontraremos esta misma distribución mimificada 👥

¿Y qué Tests tendremos **en la carpeta apps**?: Aquí meteremos **tests de aceptación** End-to-End (Aquí es donde irán nuestros tests con Behat). Para saber más sobre los distintos tipos de tests y qué capas de nuestra aplicación deben abarcar os recomendamos el curso de [Testing: introducción y buenas prácticas](https://pro.codely.tv/library/testing-introduccion-y-buenas-practicas/90916/about/)

Dentro del directorio src vamos a encontrar una carpeta por cada uno de nuestros Bounded Context, pero además tendremos el **Shared Kernel** donde recogeremos tanto Infraestructura como Dominio compartido por los diferentes contextos (por ejemplo el value object `UserId` o la clase `DBConnection`), dentro de esta carpeta Shared no tendremos en ningún momento nada correspondiente a la capa de Aplicación

![directorioSrc](directorioSrc.png "directorioSrc")

Si nos adentramos un poco más en cualquiera de los contextos que tenemos en src lo que veremos será la materialización de los Módulos que se incluían dentro de ese Bounded Context así como una carpeta Shared… Este mapeo, que se deriva en una modificación de la estructura básica de un proyecto Symfony nos permite que dicha estructura hable más de los conceptos de nuestro dominio. Además, nos permitirá identificar con más facilidad dónde añadir o modificar features y facilitar la interacción entre Módulos y su posible promoción a Bounded Contexts

A su vez, dentro de cada Módulo encontraremos siempre la misma estructura de subcarpetas (al igual que en su respectiva mimificación en la carpeta tests):

*   Application
*   Domain
*   Infrastructure

Si a nivel de apps añadíamos los tests de aceptación, los **tests unitarios de los casos de uso** (application services) irán dentro de esa carpeta Application dentro de cada Módulo. Los **tests de implementación de la infraestructura** igualmente irán dentro de la carpeta Infrastructure. Dentro de la carpeta Domain de nuestros tests lo que guardaremos serán los **Object Models** (factorías de test)

En resumen, apostamos por este approach porque

*   Nos habla mucho más sobre nuestros conceptos de dominio
*   Permite identificar con facilidad cómo interactúan los módulos entre si
*   Logramos mayor independencia entre módulos y facilitamos que éstos puedan promocionar a Bounded Context

Os dejamos aquí el [enlace](https://github.com/CodelyTV/php-ddd-skeleton/tree/0.1.0) al repositorio con todo listo para que le echéis mano 👨🏽‍💻

¿Algunda Duda?
==============

Si tenéis cualquier duda sobre el contenido de este video o queréis dejarnos cualquier sugerencia, podéis abrir una nueva discusión justo aquí abajo 👇👇👇

¡Nos vemos en la siguiente lección: 👩‍⚕️ Health check de la aplicación: Nuestro primer endpoint !