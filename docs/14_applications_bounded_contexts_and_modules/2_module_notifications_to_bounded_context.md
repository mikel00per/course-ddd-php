🆙 Promocionar el módulo notificaciones a Bounded Context
==========================================================

Hemos visto que existe una necesidad dentro de nuestro equipo de trabajo de **promocionar** el módulo de Notificaciones **a un Bounded Context** independiente de Retención. Sin embargo este tipo de promoción **no siempre implicará que nos llevemos todo lo que haya en el módulo**, es más, lo más habitual será que sólo nos llevemos algunos de los casos de uso al nuevo contexto 🗳

Gracias al modo en que venimos estableciendo la comunicación entre Módulos y Bounded Context 🗣(Por medio de CommandBus y QueryBus), lo único que tendremos que modificar será los namespaces de los servicios que movamos y la definición del CommandBus para indicarle dónde debe localizar los CommandHandlers, lo cual haremos a través de los ficheros yaml de configuración (ahora si que vamos a necesitar involucrar más de un fichero)

En el caso de las notificaciones podríamos diferenciar entre aquellos ‘mensajes transaccionales’ y los que forman parte de alguna ‘campaña de retención’. Así, entendemos a nivel de negocio que los mensajes transaccionales son derivados de las acciones de la propia plataforma, por lo que tiene sentido mantenerlos dentro del contexto de Mooc

Partiendo de este **lenguaje ubicuo** que se transmite desde el lado de negocio podemos establecer que un módulo dentro del nuevo Bounded Context será el de ‘Campaña’

Estructura del nuevo contexto:

    Retention (Bounded Context)
        |-Campaign (Module)
        |   |-Application
        |   |   |-WelcomeUser
        |   |   |   |-Schedule
        |   |   |   |-Trigger
        |   |   |     
        |   |   |-WelcomeUser
        |   |       |-Schedule
        |   |       |-Trigger        
        |   |
        |   |-Domain
        |   |-Infrastructure
        |
        |-Email
            |-Application
                |-SendWelcomeUserEmail


Con este planteamiento vemos un ejemplo algo más complejo de cómo no siempre la promoción a un nuevo Bounded Context se limitará a cambiar el package-name y mover el conjunto de ficheros, sino que este tipo de cambios implica además la toma de decisiones en cuanto a cómo vamos a querer que funcione el equipo para dicho contexto. En nuestro caso podríamos haber optado por definir un módulo por cada tipo de campaña, pero no nos convence tanto ya que puede desembocar en una gran cantidad de módulos que se nos acabaría llendo de las manos, por lo que apostamos por añadir un nivel más de identación dentro de los casos de uso

Tal como veíamos en lecciones anteriores, empujaremos la responsabilidad de saber a quién y con qué características enviar las notificaciones a los subscriptores (Recordemos 👉 OCP), que estarán definidos dentro de los distintos módulos de Email, MobilePush, Sms…

¿Alguna Duda?
=============

Si tienes alguna duda sobre este video o quieres compartir alguna sugerencia no dudes en abrir una nueva discusión más abajo 👇👇👇

¡Nos vemos en el siguiente video: 🐣 Dando de alta una nueva aplicación: Backoffice Frontend!