🐣 Dando de alta una nueva aplicación: Backoffice Frontend
===========================================================

Partimos de la necesidad de crear un frontal web para nuestro Backoffice, que implicará la creación de una nueva aplicación

*   Backoffice Backend: Llevará la parte ‘core’ junto a sus consumidores
*   Backoffice Frontend: Llevará la parte visual (maquetación, javascript…)

Es importante detallar que **cada aplicación se ocupará de contener sus propios consumidores** en su aplicación de backend para evitar precisamente que acabemos teniendo una aplicación separada de consumidores que acabe convirtiéndose en un cajón desastre difícilmente escalable y mantenible 🤯

Seguiremos el mismo esquema que veníamos utilizando para el caso de Mooc, creando una nueva carpeta ‘frontend’ dentro de /apps/backoffice con un arbol de directorios muy similar al que veíamos en /mooc/backend (nuestro frontend para el backoffice también será una aplicación Symfony)

    apps
      |-backoffice
      |   |-backend
      |   |-frontend
      |       |-bin (binarios)
      |       |-config (ficheros de configuración)
      |       |-public (assets compilados)
      |       |   |-index.php
      |       |
      |       |-src (controladores)
      |       |-var
      |
      |-mooc
          |-backend
          |-frontend


Hemos eliminado todos los ficheros y configuraciones que no necesitaremos en nuestro frontal (comandos de CQRS, dependencias, bundles…) y, por otro lado, compartiremos para esta aplicación la misma infraestructura que estábamos utilizando en Mooc 🤝 ya que de momento podremos seguir funcionando con ella sin problema (Aunque eventualmente acabará utilizando su propia infraestructura)

Al igual que para el backoffice de la plataforma, también tendremos un controlador [HealthCheck](https://github.com/CodelyTV/php-ddd-skeleton/blob/master/apps/backoffice/frontend/src/Controller/HealthCheck/HealthCheckGetController.php) que nos permitirá comprobar si la aplicación se ha levantado correctamente o si por el contrario tenemos algún problema (Otra de las ventajas que tiene Symfony es que nos ofrece una visualización muy rica de la causa del error detectado)

*   Ojo 👀, a la hora de realizar pruebas en local es posible que nos encontremos con algunos conflictos por la versión de PHP que tengamos en nuestra máquina, recordad que tenéis en el repo este [Dockerfile](https://github.com/CodelyTV/php-ddd-skeleton/blob/master/Dockerfile) que correrá con PHP 7.3.6

¿Alguna Duda?
=============

Si tienes alguna duda sobre este video o quieres compartir alguna sugerencia no dudes en abrir una nueva discusión más abajo 👇👇👇

¡Nos vemos en la siguiente lección: 🎨 Frontend web!