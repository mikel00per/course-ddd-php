🤝 Alternativas con oAuth y ACLs
================================

Aunque implementar una Basic Auth HTTP como acabamos de ver nos ofrece ciertos beneficios con muy poca integración, lo cierto es que de cara al usuario no resulta una forma de autenticación muy ‘agradable’, además de tener claras limitaciones en términos de seguridad

Una alternativa a esto sería el uso de oAuth, que nos permitiría el uso de un proveedor de autenticación externo (por ejemplo Google, Facebook, Github…) a través del cual nuestros usuarios puedan autenticarse en nuestra aplicación

![oauth](https://cdn.filestackcontent.com/YVS8Y6ySlqysszWG1qig)

El flujo que seguiría entonces nuestra petición sería el siguiente

1.  Se envía desde nuestra Aplicación al Usuario una petición de autenticación (Ej. Pantalla de Login)
2.  El usuario responde con enviando las credenciales del servicio con el que quiere autenticarse (Ej. su usuario de Google)
3.  Desde la aplicación enviamos las credenciales a la API del proveedor de autenticación para que las valide
4.  El proveedor de autenticación responde OK y devuelve un Token de acceso
5.  La aplicación a partir de este momento utiliza este token dentro de las cabeceras para solicitar recursos al backend
6.  Con este token, nuestro endpoint valida que la petición sea correcta, en cuyo caso devolverá el recurso solicitado

A nivel de código, las diferencias con la autenticación serían mínimas, ya que lo que haríamos sería simplemente contar con otro Middleware que cogiera el Token de la cabecera y lo contrastara con el servidor oAuth correspondiente

¿Alguna Duda?
=============

Si tienes alguna duda sobre este video o quieres compartir alguna sugerencia no dudes en abrir una nueva discusión más abajo 👇👇👇

¡Nos vemos en la siguiente Lección: ❓ Preguntas frecuentes!