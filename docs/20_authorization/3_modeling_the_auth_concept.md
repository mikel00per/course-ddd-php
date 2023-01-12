👌 Modelando el concepto “Auth”
===============================

Una vez implementado el Basic Auth HTTP en nuestro backend, seguimos iterando para modelar este concepto y hacerlo mucho más modular dentro de nuestra aplicación

El primer cambio que podemos identificar es que estamos pasando a esta clase el CommandBus por constructor. Puesto que en este caso **no queremos recuperar ninguna información relativa al usuario en este punto**, sino que éste se autentique, no tendría sentido utilizar una Query para este propósito. Por otro lado asumimos que el hecho de **autenticarse va a producir** algún tipo de **side effect** como ‘almacenar la última fecha de login’ que modifique nuestro universo, por lo que es un escenario donde cobra todo el sentido **lanzar un comando**

De este modo, daremos por sentado que si al lanzar el comando no se devuelve ningún error significará que todo ha ido correctamente y añadiremos los datos del usuario a la request tal y como vimos en el paso anterior. Con respecto a las excepciones devueltas, es importante puntualizar que por motivos de seguridad deberíamos devolver la misma respuesta sea cual sea el error a fin de evitar dar más detalles que pudieran facilitar debilidades a un posible atacante

*   La responsabilidad de este Middleware se limita así a la autenticación del usuario con los parámetros recibidos, si quisieramos recuperar el id u otro dato del usuario autenticado sería responsabilidad de quien lo requiera lanzar una Query al bus para solicitarlo

Hemos sacado el comando y el caso de uso al que llamará a un **nuevo módulo de Auth** dentro del contexto de Backoffice, ya que consideramos que tiene bastante sentido a nivel lógico, y en la capa de aplicación hemos creado tanto el command con su commandhandler como el caso de uso al que llamará en última instancia

Clase `UserAuthenticator`:

    final class UserAuthenticator
    {
        private $repository;
    
        public function __construct(AuthRepository $repository)
        {
            $this->repository = $repository;
        }
    
        public function authenticate(AuthUsername $username, AuthPassword $password): void
        {
            $auth = $this->repository->search($username);
            $this->ensureUserExist($auth, $username);
            $this->ensureCredentialsAreValid($auth, $password);
        }
    
        private function ensureUserExist(?AuthUser $auth, AuthUsername $username): void
        {
            if (null === $auth) {
                throw new InvalidAuthUsername($username);
            }
        }
    
        private function ensureCredentialsAreValid(AuthUser $auth, AuthPassword $password): void
        {
            if (!$auth->passwordMatches($password)) {
                throw new InvalidAuthCredentials($auth->username());
            }
        }
    }


Desde el caso de uso simplemente trataremos de recuperar el agregado `AuthUser` desde BD a partir del username y, en caso de encontrarlo, matchearemos la contraseña del agregado con la que hemos recibido en el comando. En la validación de las credenciales hacemos uso del _passwordMatches()_ del agregado AuthUser que, a su vez, llamará por debajo al propio _passwordMatches()_ del `AuthPassword`. Un siguiente paso sería la publicación de un Evento de Dominio notificando que se ha autenticado un usuario y actualizar en BD la fecha de último login

¿Alguna Duda?
=============

Si tienes alguna duda sobre este video o quieres compartir alguna sugerencia no dudes en abrir una nueva discusión más abajo 👇👇👇

¡Nos vemos en el siguiente video: 🤝 Alternativas con oAuth y ACLs!