🦐 Alternativas de cache en servidor
====================================

Existen distintas alternativas de caché a nivel de servidor que se establecen en diferentes puntos del flujo de petición en nuestra aplicación, así que veremos cada una de ellas con las ventajas e inconvenientes que cada una implica 🤷‍♀

Caché antes del Controller 👆
-----------------------------

Ya sea una pieza que se ejecute por delante del Controller (proxys reversos) o **en el propio cliente**. Ya vimos en el video anterior cómo podíamos cachear desde el cliente, en el caso de los **proxys reversos** nos encontramos con el problema de que al ser una pieza independiente no podemos ver qué es lo que se está cacheando, por otra parte nos ofrece la ventaja de tener una caché sin que tengamos que tocar ni una línea de código 🙌.

La caché por proxy reverso tiene además el beneficio de poder cachear el contenido sin que haga falta la presencia de un primer cliente. Esto es especialmente importante no sólo en contextos de alta concurrencia, sino en aquellos sitios de nuestra aplicación donde no es habitual que los usuarios entren pero que precisamente si lo harán los scrapers 🕷, y siempre querremos que éstos ‘vuelen’ dentro de nuestra página

*   Alternativa Cliente:
    *   Ejemplos: etag, expires, last-modified, cache-control
    *   Beneficio: La petición más rápida es la que no se hace 👼
*   Alternativa servidor reverse proxy:
    *   Ejemplos: Varnish, akamai, Fastly, CloudFront, Nginx, CloudFlare
    *   Beneficio: Cualquier cliente ya se beneficia (🤖)

Caché en el Controller o el Application Service ✌️
--------------------------------------------------

Otra opción sería definir un **Middleware** por delante del Application Service. este middleware reuniría toda la lógica por lo que tendríamos un único punto donde acudir a revisar cualquier cosa con la caché (con las ventajas e inconvenientes que supone esta centralización)

En nuestra opinión esta alternativa sería la menos interesante, puesto que, ni nos permitiría cachear toda la respuesta final de forma completa como en el caso anterior, ni tampoco contamos con un nivel de granularidad suficiente que nos permita jugar mejor con los contenidos concretos que queramos cachear 🤷‍♀

Otra posibilidad sería llevar la lógica de esta caché al **Application Service**, sin embargo el hecho de llevarlo a este nivel cuando no es algo que tenga que ver en absoluto con nuestra lógica de negocio (más bien con el tipo de infraestructura que estamos utilizando) nos chirría bastante por lo que tampoco estamos muy a favor de ello 🙅‍♂

Caché en el repositorio (CachedRepository Pattern) 🤟
-----------------------------------------------------

Una última alternativa sería la de llevar esta lógica a la implementación del repositorio, usando un **Decorator** de nuestro repositorio original sin que la lógica de dominio se vea afectada

La idea será entonces que cuando se haga una petición, desde este decorator compruebe primero si tiene el contenido cacheado y, en caso de no tenerlo, lo recupera de BD y lo guarda en caché antes de devolverlo

¿Alguna Duda?
=============

Si tienes alguna duda sobre este video o quieres compartir alguna sugerencia no dudes en abrir una nueva discusión más abajo 👇👇👇

¡Nos vemos en el siguiente video: 🦷 Cache de servidor: Optimizar casos de uso sin ensuciar el dominio!