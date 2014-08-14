---
layout: post
title: Aprende Symfony2 - parte 4: Controladores
tags:
    - Symfony2
    - técnico
    - serie Aprende Symfony2
---

Este es el cuarto artículo de la serie para aprender sobre
[el framework Symfony2](http://symfony.com/).
Echa un vistazo a los tres primeros:

1. {{ link('posts/2014-06-18-learn-sf2-composer-part-1.md', 'Composer') }}
2. {{ link('posts/2014-06-25-learn-sf2-empty-app-part-2.md', 'Aplicación Vacía') }}
3. {{ link('posts/2014-07-02-learn-sf2-bundles-part-3.md', 'Bundles') }}

En los anteriores artículos comenzamos a crear una aplicación vacía con los
siguientes archivos:

    .
    ├── app
    │   ├── AppKernel.php
    │   ├── cache
    │   │   └── .gitkeep
    │   ├── config
    │   │   └── config.yml
    │   └── logs
    │       └── .gitkeep
    ├── composer.json
    ├── composer.lock
    ├── src
    │   └── Knight
    │       └── ApplicationBundle
    │           └── KnightApplicationBundle.php
    ├── .gitignore
    └── web
        └── app.php

Ejecutar `composer install` debería crear el directorio `vendor`, que hemos
ignorado en git.

Aquí está el [el repositorio en el que encontrarás el código hasta ahora](https://github.com/gnugat/learning-symfony2/releases/tag/3-bundles).

En este artículo, aprenderemos más sobre el enrutamiento y los controladores.

## Conociendo el enrutamiento y los controladores

Para familiarizarnos con el enrutamiento y los controladores, crearemos una
ruta que no devuelve nada. Lo primero que hay que hacer es configurar el
router:

    # File: app/config/app.yml
    framework:
        secret: "Three can keep a secret, if two of them are dead."
        router:
            resource: %kernel.root_dir%/config/routing.yml

Ahora podemos escribir nuestras rutas en un archivo separado:

    # File: app/config/routing.yml
    what_john_snow_knows:
        path: /api/ygritte
        methods:
            - GET
        defaults:
            _controller: KnightApplicationBundle:Api:ygritte

Como ves, una ruta tiene:

* un nombre (`what_john_snow_knows`)
* a patrón (`/api/ygritte`)
* uno o más verbos HTTP (`GET`)
* un controlador `Knight\ApplicationBundle\Controller\ApiController::ygritteAction()`

*Nota*: el parámetro `_controller` es un atajo compuesto de  tres partes, que
son el nombre del bundle, después el nombre del controlador sin sufijo, y
finalmente el nombre del método sin sufijo.

Ahora necesitamos crear el siguiente directorio:

    mkdir src/Knight/ApplicationBundle/Controller

Y la siguiente clase o controlador:

    <?php
    // File: src/Knight/ApplicationBundle/Controller/ApiController.php

    namespace Knight\ApplicationBundle\Controller;

    use Symfony\Bundle\FrameworkBundle\Controller\Controller;
    use Symfony\Component\HttpFoundation\Request;
    use Symfony\Component\HttpFoundation\Response;

    class ApiController extends Controller
    {
        public function ygritteAction(Request $request)
        {
            return new Response('', Response::HTTP_NO_CONTENT);
        }
    }

Para probarlo, te recomiendo usar un cliente HTTP. Instalemos
[HTTPie, the CLI HTTP client](http://httpie.org):

    sudo apt-get install python-pip
    sudo pip install --upgrade httpie

Ya podemos probar nuestro webservice:

    http GET knight.local/api/ygritte

La primera línea debería ser `HTTP/1.1 204 No Content`.

## Posteando datos

Nuestro *scrum master* y nuestro jefe de negocio han escrito un camino para
nosotros:

    Como Caballero de Ni
    Quiero un webservice que diga "ni"
    Para conseguir una almáciga

Esto significa que vamos a necesitar la siguiente ruta:

    # File: app/config/routing.yml
    ni:
        path: /api/ni
        methods:
            - POST
        defaults:
            _controller: KnightApplicationBundle:Api:ni

Nuestro controlador recogerña el valor posteado (llamado `offering`), comprobará
si es una almáciga, y devolverá una respuesta que contenga `Ni` (en caso de
  error) o `Ecky-ecky-ecky-ecky-pikang-zoop-boing-goodem-zoo-owli-zhiv` (en
  caso de éxito):

    <?php
    // File: src/Knight/ApplicationBundle/Controller/ApiController.php

    namespace Knight\ApplicationBundle\Controller;

    use Symfony\Bundle\FrameworkBundle\Controller\Controller;
    use Symfony\Component\HttpFoundation\Request;
    use Symfony\Component\HttpFoundation\Response;
    use Symfony\Component\HttpFoundation\JsonResponse;

    class ApiController extends Controller
    {
        public function niAction(Request $request)
        {
            $postedContent = $request->getContent();
            $postedValues = json_decode($postedContent, true);

            $answer = array('answer' => 'Ecky-ecky-ecky-ecky-pikang-zoop-boing-goodem-zoo-owli-zhiv');
            $statusCode = Response::HTTP_OK;
            if (!isset($postedValues['offering']) || 'shrubbery' !== $postedValues['offering']) {
                $answer['answer'] = 'Ni';
                $statusCode = Response::HTTP_UNPROCESSABLE_ENTITY;
            }

            return new JsonResponse($answer, $statusCode);
        }
    }

La clase `JsonResponse` convertirá el array a JSON y establecerá los headers
HTTP correctos.

Si intentamos enviar algo sospechoso, como esto:

    http POST knight.local/api/ni offering=arenque

Deberíamos obtener una respuesta parecida a:

    HTTP/1.1 422 Unprocessable Entity
    Cache-Control: no-cache
    Content-Type: application/json
    Date: Thu, 10 Jul 2014 15:23:00 GMT
    Server: Apache
    Transfer-Encoding: chunked

    {
        "answer": "Ni"
    }

Y cuando enviemos la ofrenda correcta:

    http POST knight.local/api/ni offering=shrubbery

Deberñiamos obtener algo similar a:

    HTTP/1.1 200 OK
    Cache-Control: no-cache
    Content-Type: application/json
    Date: Thu, 10 Jul 2014 21:42:00 GMT
    Server: Apache
    Transfer-Encoding: chunked

    {
        "answer": "Ecky-ecky-ecky-ecky-pikang-zoop-boing-goodem-zoo-owli-zhiv"
    }

## API de la clase Request

Esta es parte de la API de la clase Request:

    <?php

    namespace Symfony\Component\HttpFoundation;

    class Request
    {
        public $request; // Request body parameters ($_POST)
        public $query; // Query string parameters ($_GET)
        public $files; // Uploaded files ($_FILES)
        public $cookies; // $_COOKIE
        public $headers; // Taken from $_SERVER

        public static function createFromGlobals():
        public static function create(
            $uri,
            $method = 'GET',
            $parameters = array(),
            $cookies = array(),
            $files = array(),
            $server = array(),
            $content = null
        );

        public function getContent($asResource = false);
    }

Usamos `createFromGlobals` en nuestro controlador frontal (`web/app.php`), y
hace exactamente lo que dice: inicializa la Request desde las variables super
globales de PHP (`$_POST`, `$_GET`, etc).

El método `create` es realmente útil en tests, dado que no necesitaremos
sobreescribir los valores de las variables super globales de PHP.

Todos los atributos que aparecen listados son instancias de la clase
`Symfony\Component\HttpFoundation\ParameterBag`, que es como un array orientado
a objetos, con los métodos `set`, `has` y `get` (entre otros).

Cuando envías un formulario, tu navegador establece automáticamente el parámetro
 `Content-Type`del header de la petición HTTP a
 `application/x-www-form-urlencoded`, y los valores del formulario son enviados
 en el contenido  de la peticion de esta forma:

    offering=arenque

PHP entiende esta petición, y pondrá los valores en la variable super global
`$_POST`. Esto significa que podrás recuperarlos así:

    $request->request->get('offering');

De todos modos, cuando enviamos algo en JSON con el `Content-Type`
`application/json`, PHP no rellena `$_POST`. Tendrás que recuperar los datos en
crudo con `getContent` y convertirlos usando `json_decode`, como hemos hecho en
nuestro controlador.

## API de la clase Response

Esta es parte de la API de la clase Response:

    <?php

    namespace Symfony\Component\HttpFoundation;

    class Response
    {
        const HTTP_OK = 200;
        const HTTP_CREATED = 201;
        const HTTP_NO_CONTENT = 204;
        const HTTP_UNAUTHORIZED = 401;
        const HTTP_FORBIDDEN = 403;
        const HTTP_NOT_FOUND = 404;
        const HTTP_UNPROCESSABLE_ENTITY = 422; // RFC4918

        public $headers; // @var Symfony\Component\HttpFoundation\ResponseHeaderBag

        public function __construct($content = '', $status = 200, $headers = array())

        public function getContent();
        public function getStatusCode();

        public function isSuccessful();
    }

Hay muchas constantes de códigos de estado HTTP, así que he seleccionado
solamente los que más uso.

Puedes establecer y recuperar los headers de Response's vía una propiedad
pública, que también es un `ParameterBag`.

El constructor te permite establecer el contenido, el código de estado y los
headers.

Los otros tres métodos se usan sobre todo en tests. Hay muchos métodos `is` para
comprobar el tipo de petición, pero normalmente lo único que te interesará será
saber que la respuesta es satisfactoria.

Otros tipos de respuesta:

* `JsonResponse`: establece el `Content-Type` y convierte el contenido a JSON
* `BinaryFileResponse`: establece los headers y adjunta un archivo a la
 respuesta
* `RedirectResponse`: establece el destino para una redirección
* `StreamedResponse`: útil para el streaming de archivos muy grandes

## Conclusión

Symfony2 es un framework HTTP, cuyas principales API son los controladores:
reciben como parámetro una petición y devuelven una respuesta. Todo lo que
tenemos que hacer es crear un controlador, escribir una mínima configuración
para enlazarlo a una URL ¡y estaremos listos!

No olvides subir tu trabajo al repositorio:

    git add -A
    git commit -m 'Creada la ruta Ni y el controlador'

El próximo artículo debería tratar sobre tests: ¡permanece atento!

### Próximos artículos

* {{ link('posts/2014-07-20-learn-sf2-tests-part-5.md', '5: Tests') }}
* {{ link('posts/2014-07-23-learn-sf2-conclusion.md', 'Conclusión') }}

### Artículos anteriores

* {{ link('posts/2014-06-18-learn-sf2-composer-part-1.md', '1: Composer') }}
* {{ link('posts/2014-06-25-learn-sf2-empty-app-part-2.md', '2: Aplicación Vacía') }}
* {{ link('posts/2014-07-02-learn-sf2-bundles-part-3.md', '3: Bundles') }}
