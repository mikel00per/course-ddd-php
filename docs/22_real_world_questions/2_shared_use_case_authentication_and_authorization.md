Caso de uso compartido: Autenticación y Autorización
====================================================

Otra cuestión que nos puede surgir al aplicar DDD es qué hacemos cuando tenemos casos de uso compartido por distintos Bounded Contexts ¿Deberíamos tener estos casos de uso en el Shared en lugar de en un contexto a parte? 🤝

Un ejemplo de esto sucede con la autenticación y autorización 🔒 de los usuarios para llevar a cabo distintas acciones en varios contextos de nuestra aplicación

Hay que tener en cuenta que una de las razones por las que usamos Bounded Contexts es **separar responsabilidades** y, en el momento en que esta lógica de Auth comience a crecer y sea necesario sacarla fuera, será un problema establecer de quien es la responsabilidad de mantenerlo, además de que usará **su propia infraestructura de BBDD**, algo que, en nuestra opinión, no debería estar en el contexto de Shared

En general, y salvo pequeñas excepciones, el hecho de mantener partes de infraestructura en Shared es un code smell a nivel de macro-diseño y deberíamos evitar aquellas aproximaciones que nos puedan llevar a este escenario 👃

En este caso, nuestra **propuesta** sería **definir un contexto propio de Auth** con sus VO, casos de uso, infraestructura propia… Y añadir un **Middleware delante de los Controllers** de nuestras aplicaciones que se comunique con este contexto vía Command/Query para hacer login y pasar a los Controllers los datos necesarios para llevar a cabo la validación de permisos

Es importante señalar aquí que este contexto de Auth no debería ser el responsable de gestionar los permisos de todos los casos de uso de cada contexto que lo invoca, sino que debe ser cada servicio de aplicación el responsable de conocer quien puede o no realizar la acción 👮

¿Alguna Duda?
=============

Si tienes alguna duda sobre el contenido del video o quieres compartir alguna sugerencia no dudes en abrir una nueva discusión más abajo 👇👇👇

¡Nos vemos en el siguiente video: Relación entre módulos User y Auth!