✋ Separando el contexto de mooc de backoffice
=============================================

Si echamos la mirada atrás, al inicio del curso planteábamos un diseño con dos Bounded Contexts diferenciados (Mooc y Backoffice) dentro de cada uno de los cuales queríamos separar las aplicaciones de Frontend y Backend. Puesto que conforme hemos ido avanzando hemos ido **generando cierta deuda técnica** y ‘ensuciado’ nuestros contextos es momento de limpiar y poner cierto orden 👮‍♂️. En concreto hemos introducido la **infraestructura de Mooc dentro de Backoffice**

Definición de servicios previa en `services.yaml` de Mooc-Backend:

      CodelyTv\:
        resource: '../../../../src/'


Tal como veníamos definiendo la configuración en el ‘services.yaml’ de cada aplicación, lo que hacíamos era establecer como servicios todo lo que se encontrase dentro del directorio _src_ con el namespace de \`CodelyTv’, es decir, estábamos definiendo implícitamente como servicios lo relativo a todos los contextos además del Shared.

Definición de servicios actual en `services.yaml` de Mooc-Backend:

      CodelyTv\Shared\:
        resource: '../../../../src/Shared'
    
      CodelyTv\Mooc\:
        resource: '../../../../src/Mooc'


De este modo evitamos traernos aquellas dependencias que no necesitamos (o no deberíamos necesitar) además de hacer explícitos los servicios que nuestra aplicación está utilizando y restringiendo así las llamadas a Queries/Commands que no sean del propio contexto

En el caso del contexto de Backoffice Frontend vemos como, si intentamos no incluir la carpeta de Mooc entre los servicios a poder inyectar nos dará un error al no encontrar la Query que ‘tomamos’ del contexto de Mooc. No obstante, de momento la estrategia será ir pasito a pasito y asumir esta dependencia como algo que solucionaremos más adelante, por lo que el cambio será simplemente hacer explícita esta condición

Definición de servicios actual en `services.yaml` de Backoffice-Frontend:

      CodelyTv\Shared\:
        resource: '../../../../src/Shared'
    
    CodelyTv\Backoffice\:
        resource: '../../../../src/Backoffice'
    
      CodelyTv\Mooc\:
        resource: '../../../../src/Mooc'


Puesto que aunque de forma explícita nosotros solo estamos llamando desde el contexto de Backoffice a la `FindCoursesCounterQuery`, esta a su vez requerirá otra serie de dependencias de su propio contexto por lo que tratar de inyectar únicamente esta clase en el contexto de Backoffice no es tampoco una solución mejor, de modo que nos traemos todo sin filtrar

¿Alguna Duda?
=============

Si tienes alguna duda sobre este video o quieres compartir alguna sugerencia no dudes en abrir una nueva discusión más abajo 👇👇👇

¡Nos vemos en el siguiente video: 🤚 Extraer aplicación backoffice backend!