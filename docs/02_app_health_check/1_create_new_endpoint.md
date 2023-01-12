👩‍⚕️ Creando un nuevo endpoint
===============================

Un Health check tiene como objetivo comprobar si un servidor está bien desplegado.

En los cursos de [AWS EC2: Tu primer deploy](https://pro.codely.tv/library/aws-deploy-en-ec2/62577/about/), y [AWS: Escalando apps - Load Balancers y Auto Scaling Groups](https://pro.codely.tv/library/aws-autoescalado-de-aplicaciones-con-alb-y-asg/66588/about/) vimos cómo utilizar este tipo de endpoints para el despliegue de nuestra aplicación

Pero ojo! 👀 no debemos confundir este tipo de endpoint con los endpoints de ‘status’, que comprobarán además de la salud del servidor, si otros sistemas implicados (como el acceso a BD) también están funcionando correctamente.

**CodelyTv Tip** ☝️: Si lo que queremos es comprobar si un servidor funciona o no para saber si debemos ‘matar’ la máquina en la que se encuentra y levantar una nueva, debemos evitar los endpoints de status, puesto que si lo que realmente falla es la conexión a BD, entraeremos en un bucle continuo echando abajo y levantando nuevas máquinas sin que se solucione el verdadero problema

Recordad que para crear el proyecto en su estado inicial tal y como vemos en este video, debemos indicar en el comando la versión 0.4

> composer create-project codelytv/ddd-skeleton:0.4.0

Clase `HealthCheckGetController`:

    use Symfony\Component\HttpFoundation\JsonResponse;
    use Symfony\Component\HttpFoundation\Request;
    use Symfony\Component\HttpFoundation\Response;
    
    final class HealthCheckGetController
    {
        private $generator;
        public function __construct(RandomNumberGenerator $generator)
        {
            $this->generator = $generator;
        }
        public function __invoke(Request $request): Response
        {
            return new JsonResponse(
                [
                    'status' => 'ok'
                ]
            );
        }
    }


Crearemos nuestros endpoints dentro del directorio apps/mooc/backend/src/Controller/ de forma que generaremos una carpeta para cada concepto. Siguiendo los principios que vimos en el curso de [Arquitectura Hexagonal](https://pro.codely.tv/library/arquitectura-hexagonal/66748/about/), especificaremos el método GET en el propio nombre del controlador porque queremos que la propia clase denote su razón de ser, es decir, va a recibir peticiones GET por Http y devolverá respuestas por Http. Aprovecharemos la funcionalidad de `JsonResponse` de Symfony para que nos genere todo lo necesario para que la respuesta esté bien implementada

Enrutando el nuevo controller 🚊
--------------------------------

Dentro de Symfony, solemos encontrar dos formas de definir el enrutamiento de los controladores: anotaciones y ficheros yaml. En nuestro caso hemos escogido el uso de ficheros porque además de mantener más limpios los controladores, nos permite tener todo el sistema de routing mucho más centralizado. Si accedemos al directorio routes encontraremos un fichero yaml por cada concepto

Fichero `health_check.yaml`:

    health-check_get:
        path: /health-check
        controller: CodelyTv\Apps\Mooc\Backend\Controller\HealthCheck\HealthCheckGetController
        methods:  [GET]


la estructura de las rutas en Symfony se compone de un _path_ por el que entraría, el _controlador_ con el fullname de la ruta al que se dirige y el _método_ http permitido.

Si hubieramos definido con otro nombre de función nuestro controlador, deberíamos haberselo especificado mediante un `::action` al final de la clave ‘controller’, sin embargo, con al definirlo como una función `_invoke()`, Symfony sabe que debe debe llamar a esta función cuando no se especifica nada (Esto nos ayuda además a que el controlador sólo tenga un punto de entrada y por tanto haga una sola cosa)

una vez añadido, sólo nos queda comprobar que está funcionando correctamente, y para ello levantaremos el servidor local de desarrollo con el comando `make start-local` que hemos preparado en el fichero Makefile y llamaremos al punto de entrada

    curl localhost:8090/health-check


# ¿Alguna Duda?

Seguimos en el siguiente video adentrándonos en cómo está funcionando por dentro nuestro endpoint y conociendo las magias detras del framework con el que estamos trabajando. Recuerda que puedes abrir una discusión aquí abajo 👇 Si tienes alguna duda o te gustaría compartir cualquier sugerencia con nosotros ¡Nos vemos!