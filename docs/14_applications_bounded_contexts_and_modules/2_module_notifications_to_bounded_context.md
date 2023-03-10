馃啓 Promocionar el mo虂dulo notificaciones a Bounded Context
==========================================================

Hemos visto que existe una necesidad dentro de nuestro equipo de trabajo de **promocionar** el m贸dulo de Notificaciones **a un Bounded Context** independiente de Retenci贸n. Sin embargo este tipo de promoci贸n **no siempre implicar谩 que nos llevemos todo lo que haya en el m贸dulo**, es m谩s, lo m谩s habitual ser谩 que s贸lo nos llevemos algunos de los casos de uso al nuevo contexto 馃棾

Gracias al modo en que venimos estableciendo la comunicaci贸n entre M贸dulos y Bounded Context 馃棧(Por medio de CommandBus y QueryBus), lo 煤nico que tendremos que modificar ser谩 los namespaces de los servicios que movamos y la definici贸n del CommandBus para indicarle d贸nde debe localizar los CommandHandlers, lo cual haremos a trav茅s de los ficheros yaml de configuraci贸n (ahora si que vamos a necesitar involucrar m谩s de un fichero)

En el caso de las notificaciones podr铆amos diferenciar entre aquellos 鈥榤ensajes transaccionales鈥? y los que forman parte de alguna 鈥榗ampa帽a de retenci贸n鈥?. As铆, entendemos a nivel de negocio que los mensajes transaccionales son derivados de las acciones de la propia plataforma, por lo que tiene sentido mantenerlos dentro del contexto de Mooc

Partiendo de este **lenguaje ubicuo** que se transmite desde el lado de negocio podemos establecer que un m贸dulo dentro del nuevo Bounded Context ser谩 el de 鈥楥ampa帽a鈥?

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


Con este planteamiento vemos un ejemplo algo m谩s complejo de c贸mo no siempre la promoci贸n a un nuevo Bounded Context se limitar谩 a cambiar el package-name y mover el conjunto de ficheros, sino que este tipo de cambios implica adem谩s la toma de decisiones en cuanto a c贸mo vamos a querer que funcione el equipo para dicho contexto. En nuestro caso podr铆amos haber optado por definir un m贸dulo por cada tipo de campa帽a, pero no nos convence tanto ya que puede desembocar en una gran cantidad de m贸dulos que se nos acabar铆a llendo de las manos, por lo que apostamos por a帽adir un nivel m谩s de identaci贸n dentro de los casos de uso

Tal como ve铆amos en lecciones anteriores, empujaremos la responsabilidad de saber a qui茅n y con qu茅 caracter铆sticas enviar las notificaciones a los subscriptores (Recordemos 馃憠 OCP), que estar谩n definidos dentro de los distintos m贸dulos de Email, MobilePush, Sms鈥?

驴Alguna Duda?
=============

Si tienes alguna duda sobre este video o quieres compartir alguna sugerencia no dudes en abrir una nueva discusi贸n m谩s abajo 馃憞馃憞馃憞

隆Nos vemos en el siguiente video: 馃悾 Dando de alta una nueva aplicacio虂n: Backoffice Frontend!