馃 Extraer aplicaci贸n backoffice backend
========================================

Seguimos resolviendo la deuda t茅cnica que hab铆amos generado previamente y ahora le llega el turno a la API que hab铆amos creado dentro del Backoffice Frontend y que deber铆a estar en una aplicaci贸n propia de Backoffice Backend

Aunque ya hemos visto durante el curso el proceso de creaci贸n de una nueva aplicaci贸n desde 0 馃懚, algo a lo que no nos hab铆amos enfrentado a煤n era al hecho de extraer una porci贸n de c贸digo de una aplicaci贸n ya existente 馃懆

En este caso hemos optado por una 鈥榲铆a r谩pida鈥? copiando todo el contenido de la aplicaci贸n frontend y borrando todo lo que no se quedar谩 en la aplicaci贸n backend

Desde el 鈥楻eplace in Path鈥? del IDE cambiamos todas las ocurrencias de frontend/Frontend dentro del directorio de /backoffice/backend/ para corregir los namespaces de las clases

Una vez corregidos los namespaces borramos las templates, im谩genes y el resto de ficheros que requer铆a el frontal adem谩s de los Controladores a excepci贸n de la API (aunque si que tendremos que eliminar de 茅sta la herencia de `Controller`) y el HealthCheck y las rutas que no vayamos a requerir, que hemos modificado para hacer referencia a esta nueva aplicaci贸n (Ojo 馃憖 por otra parte tendremos que borrar el `ApiCoursesGetController` del frontend y modificar all铆 donde era llamado)

Dentro del `services.yaml` tambi茅n podremos eliminar la dependencia al contexto de Mooc

Finalmente tendremos que a帽adir esta nueva aplicaci贸n en los start-local y stop-local del fichero `Makefile` para poder levantarlo con el resto de aplicaciones

Al extraer esta API a una aplicaci贸n aparte que corre en un puerto distinto nos vamos a encontrar con el cl谩sico problema de pol铆tica de CORS, ante esto optamos por solucionarlo a帽adiendo en los headers el atributo 鈥楢ccess-Control-Allow-Origin鈥? permitiendo peticiones desde cualquier dominio

驴Alguna Duda?
=============

Si tienes alguna duda sobre este video o quieres compartir alguna sugerencia no dudes en abrir una nueva discusi贸n m谩s abajo 馃憞馃憞馃憞

隆Nos vemos en la siguiente lecci贸n: 鉁? Optimizando rendimiento - M谩s all谩 de Elasticsearch: Cache de cliente y servidor!