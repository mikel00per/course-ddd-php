🤚 Extraer aplicación backoffice backend
========================================

Seguimos resolviendo la deuda técnica que habíamos generado previamente y ahora le llega el turno a la API que habíamos creado dentro del Backoffice Frontend y que debería estar en una aplicación propia de Backoffice Backend

Aunque ya hemos visto durante el curso el proceso de creación de una nueva aplicación desde 0 👶, algo a lo que no nos habíamos enfrentado aún era al hecho de extraer una porción de código de una aplicación ya existente 👨

En este caso hemos optado por una ‘vía rápida’ copiando todo el contenido de la aplicación frontend y borrando todo lo que no se quedará en la aplicación backend

Desde el ‘Replace in Path’ del IDE cambiamos todas las ocurrencias de frontend/Frontend dentro del directorio de /backoffice/backend/ para corregir los namespaces de las clases

Una vez corregidos los namespaces borramos las templates, imágenes y el resto de ficheros que requería el frontal además de los Controladores a excepción de la API (aunque si que tendremos que eliminar de ésta la herencia de `Controller`) y el HealthCheck y las rutas que no vayamos a requerir, que hemos modificado para hacer referencia a esta nueva aplicación (Ojo 👀 por otra parte tendremos que borrar el `ApiCoursesGetController` del frontend y modificar allí donde era llamado)

Dentro del `services.yaml` también podremos eliminar la dependencia al contexto de Mooc

Finalmente tendremos que añadir esta nueva aplicación en los start-local y stop-local del fichero `Makefile` para poder levantarlo con el resto de aplicaciones

Al extraer esta API a una aplicación aparte que corre en un puerto distinto nos vamos a encontrar con el clásico problema de política de CORS, ante esto optamos por solucionarlo añadiendo en los headers el atributo ‘Access-Control-Allow-Origin’ permitiendo peticiones desde cualquier dominio

¿Alguna Duda?
=============

Si tienes alguna duda sobre este video o quieres compartir alguna sugerencia no dudes en abrir una nueva discusión más abajo 👇👇👇

¡Nos vemos en la siguiente lección: ✨ Optimizando rendimiento - Más allá de Elasticsearch: Cache de cliente y servidor!