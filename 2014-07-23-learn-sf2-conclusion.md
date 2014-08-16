---
layout: post
title: Aprende Symfony2 - Conclusión
tags:
    - Symfony2
    - técnico
    - serie Aprende Symfony2
---

Este es el resumen de la serie para aprender sobre
[el framework Symfony2](http://symfony.com/).
Echa un vistazo a los artículos que la componen:

1. {{ link('posts/2014-06-18-learn-sf2-composer-part-1.md', 'Composer') }}
2. {{ link('posts/2014-06-25-learn-sf2-empty-app-part-2.md', 'Aplicación Vacía') }}
3. {{ link('posts/2014-07-02-learn-sf2-bundles-part-3.md', 'Bundles') }}
4. {{ link('posts/2014-07-12-learn-sf2-controllers-part-4.md', 'Controladores') }}
5. {{ link('posts/2014-07-20-learn-sf2-tests-part-5.md', 'Tests') }}

En los anteriores artículos una aplicación testeada para Los Caballeros Que
Dicen 'Ni' con los siguientes archivos:

    .
    ├── app
    │   ├── AppKernel.php
    │   ├── cache
    │   │   └── .gitkeep
    │   ├── config
    │   │   ├── config_test.yml
    │   │   ├── config.yml
    │   │   └── routing.yml
    │   ├── logs
    │   │   └── .gitkeep
    │   └── phpunit.xml.dist
    ├── composer.json
    ├── composer.lock
    ├── src
    │   └── Knight
    │       └── ApplicationBundle
    │           ├── Controller
    │           │   └── ApiController.php
    │           ├── KnightApplicationBundle.php
    │           └── Tests
    │               └── Controller
    │                   └── ApiControllerTest.php
    ├── .gitignore
    └── web
        └── app.php

Ejecutar `composer install` debería crear el directorio `vendor`, que hemos
ignorado en git.

Aquí está el [el repositorio en el que encontrarás el código hasta ahora](https://github.com/gnugat/learning-symfony2/tree/5-tests).

Este artículo será una *cheat sheet* de todo lo visto en la serie.

## Composer

[Composer](https://getcomposer.org/) te ayuda a instalar y actualizar librerías
de terceros.

Descárgalo una sola vez e instálalo en los binarios globales de tu SO:

    curl -sS https://getcomposer.org/installer | php
    sudo mv ./composer.phar /usr/local/bin/composer

Deberías poder ejecutarlo así: `composer`.

* instalar una librería de terceros: `composer require [--dev] <vendor/name:version>`
* descargar las librerías de terceros del proyecto: `composer install`
* actualizar las librerías de terceros del proyecto: `composer update`

Las librerías de terceros disponibles se pueden encontrar en
[Packagist](https://packagist.org/).

Aquí tienes la
[explicación de Igor sobre las constraints de versiones en Composer](https://igor.io/2013/01/07/composer-versioning.html).

En estos artículos hemos creado un proyecto desde cero, pero la manera
recomendada de comenzar una aplicación Symfony2 es usar el comando de
inicialización de Composer:
`composer create-project <vendor/name> <path-to-install>`

Puedes usar la [Symfony Standard Edition](https://github.com/symfony/symfony-standard)
(`symfony/framework-standard-edition`), o cualquier otra distribución.

Te aconsejo que utilices una distribución vacía como
[Symfony Empty Edition](https://github.com/gnugat/symfony-empty):

    composer create-project gnugat/symfony-framework-empty-edition <path-to-install>

*Consejo*: En el servidor de producción, usa este comando para instalar las
dependencias del proyecto (las librerías de terceros):

    composer install --no-dev --optimize

## Bundles

Integran tu código con el framework. Más concretamente, configuran el contenedor
de inyección de dependencias del kernel.

*Nota*: Para saber  más sobre Inyección de Dependencias, echa un vistazo a los
siguientes artículos:

* {{ link('posts/2014-01-22-ioc-di-and-service-locator.md', 'Inversion of Control, Dependency Injection, Dependency Injection Container and Service Locator') }}
* {{ link('posts/2014-01-29-sf2-di-component-by-example.md', 'Symfony2 Dependency Injection component, by example') }}

El único bundle que necesitas crear es el `ApplicationBundle`, donde estará
todo tu código. Así es como se hace:

1. crea su directorio: `mkdir -p src/<Vendor>/<Name>Bundle`
2. crea su clase: `$EDITOR src/<Vendor>/<Name>Bundle/<Vendor><Name>Bundle.php`
3. regístralo en el kernel: `$EDITOR app/AppKernel.php`

Una clase Bundle tiene esta pinta:

    <?php
    // File: src/Knight/ApplicationBundle/KnightApplicationBundle.php

    namespace Knight\ApplicationBundle;

    use Symfony\Component\HttpKernel\Bundle\Bundle;

    class KnightApplicationBundle extends Bundle
    {
    }

## Aplicación

En tu aplicación, hay muy pocos archivos relacionados con el framework
Symfony2. Esta es la lista de los que editarás normalmente:

### El kernel de la aplicación

El archivo `app/AppKernel.php` es donde los bundles se registran y donde se
carga la configuración. Sólo necesitarás editarlo cuando instales un bundle
nuevo.

Manera de proceder: primero instala el bundle vía Composer:

    composer require [--dev] <vendor/name:version>

Después regístralo en el kernel de la aplicación:

    <?php
    // File: app/AppKernel.php

    use Symfony\Component\HttpKernel\Kernel;
    use Symfony\Component\Config\Loader\LoaderInterface;

    class AppKernel extends Kernel
    {
        public function registerBundles()
        {
            $bundles = array(
                new Symfony\Bundle\FrameworkBundle\FrameworkBundle(),
                new Symfony\Bundle\SecurityBundle\SecurityBundle(),
                new Symfony\Bundle\TwigBundle\TwigBundle(),
                new Symfony\Bundle\MonologBundle\MonologBundle(),
                new Symfony\Bundle\AsseticBundle\AsseticBundle(),
                new Doctrine\Bundle\DoctrineBundle\DoctrineBundle(),
                new Sensio\Bundle\FrameworkExtraBundle\SensioFrameworkExtraBundle(),

                // Añade aquí tus bundles
            );

            if (in_array($this->getEnvironment(), array('dev', 'test'))) {
                $bundles[] = new Symfony\Bundle\WebProfilerBundle\WebProfilerBundle();
                $bundles[] = new Sensio\Bundle\DistributionBundle\SensioDistributionBundle();
                $bundles[] = new Sensio\Bundle\GeneratorBundle\SensioGeneratorBundle();

                // O aquí, si quieres que sólo estén disponibles en el entorno de testing
            }

            return $bundles;
        }

        public function registerContainerConfiguration(LoaderInterface $loader)
        {
            $loader->load(__DIR__.'/config/config_'.$this->getEnvironment().'.yml');
        }
    }

### La configuración del enrutamiento

El archivo `app/config/routing.yml` es donde enlazarás la acción de un
controlador a una URL. Ejemplo:

    # File: app/config/routing.yml
    ni:
        path: /api/ni
        methods:
            - POST
        defaults:
            _controller: KnightApplicationBundle:Api:ni

    question_to_cross_the_bridge:
        path: /api/question/{number}
        methods:
            - GET
        defaults:
            _controller: KnightApplicationBundle:Api:question

Como ves, puedes definir las rutas usando *placeholders* (o comodines), que
después estarán disponibles en el controlador vía el objeto Request:

    $request->query->get('number'); // query is an instance of ParameterBag

### Controladores, tu punto de entrada

Cada ruta se asocia a la acción de un controlador.

Un controlador es una clase localizada en
`src/<Vendor>/ApplicationBundle/Controller`,
con el sufijo `Controller`.

Una acción es un método público de un controlador, con el sufijo `Action`,
que recibe un parámetro `Request $request` y debe devolver una instancia del
objeto `Response`:

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

*Nota*: puedes crear sub-directorios en
`src/<Vendor>/ApplicationBundle/Controller`, permitiéndote así categorizar tus
controladores. En la definición de tus rutas, se vería así:
`KnightApplicationBundle:Subdirectory\Controller:action`.

### Tests funcionales

Puedes usar cualquier framework de testing en un proyecto Symfony2. PHPUnit es
uno de ellos, y muy popular, por lo que es el que usamos en nuestros ejemplos.

Los tests funcionales replican los controladores y comprueban si el código de
estado es correcto. Si estás construyendo una API, puedes comprobar en mayor
profundidad si el código de estado es el esperado:

    <?php
    // File: src/Knight/ApplicationBundle/Tests/Controller/ApiControllerTest.php

    namespace Knight\ApplicationBundle\Tests\Controller;

    use Symfony\Bundle\FrameworkBundle\Test\WebTestCase;
    use Symfony\Component\HttpFoundation\Response;

    class ApiControllerTest extends WebTestCase
    {
        private function post($uri, array $data)
        {
            $content = json_encode($data);
            $client = static::createClient();
            $client->request('POST', $uri, array(), array(), array(), $content);

            return $client->getResponse();
        }

        public function testOfferingTheRightThing()
        {
            $response = $this->post('/api/ni', array('offering' => 'shrubbery'));

            $this->assertSame(Response::HTTP_OK , $response->getStatusCode());
        }

        public function testOfferingTheWrongThing()
        {
            $response = $this->post('/api/ni', array('offering' => 'hareng'));

            $this->assertSame(Response::HTTP_UNPROCESSABLE_ENTITY , $response->getStatusCode());
        }
    }

La clase `WebTestCase` nos la proporciona el framework: crea una aplicación
(como hacemos nosotros en `web/app.php`), por lo que puedes enviar peticiones y
testear las respuestas.

### Dónde poner tu propio código

Puedes poner tu código en cualquier punto de `src/<Vendor>/ApplicationBundle`.

¿Quién dijo que debes desacoplar tu código de Symfony2? ¡Puedes escribirlo
desacoplado directamente!

La convención es crear directorios con nombres relativos a los objetos que
contienen. Por ejemplo, el directorio `Controller` contiene clases controladoras
(a las que se añade el sufijo `Controller`). No tienes por qué seguir estas
convenciones (salvo para los controladores y los comandos): ¡Organízate como
prefieras!

## Conclusión

Symfony2 se aparta de tu camino, las únicas clases del framework que necesitamos
usar son el controlador, la petición y la respuesta.

El flujo de trabajo es realmente simple:

1. Symfony2 convierte la petición HTTP en el objeto `Request`
2. el componente de routing permite que se ejecute el controlador relacionado
con la URL actual
3. el controlador recibe el objeto `Request` como parámetro, y debe devolver
 un objeto `Response`
4. Symfony2 convierte el objeto `Response` en la respuesta HTTP

### Qué debería hacer a continuación?

Practicar.

Sabes lo estrictamente necesario sobre Symfony2, y la única manera de aprender
más es practicar, encontrar nuevos casos de uso, encontrar respuestas en la
[documentación](http://symfony.com/doc/current/index.html) y preguntar en
[StackOverflow](http://stackoverflow.com/questions/tagged/symfony2) (si nadie
  lo ha preguntado antes).

Si realmente quieres dominar Symfony2, permanece atento: ¡Escribiré una nueva
serie de artículos!

### Artículos previos

* {{ link('posts/2014-06-18-learn-sf2-composer-part-1.md', '1: Composer') }}
* {{ link('posts/2014-06-25-learn-sf2-empty-app-part-2.md', '2: Aplicación vacía') }}
* {{ link('posts/2014-07-02-learn-sf2-bundles-part-3.md', '3: Bundles') }}
* {{ link('posts/2014-07-12-learn-sf2-controllers-part-4.md', '4: Controladores') }}
* {{ link('posts/2014-07-20-learn-sf2-tests-part-5.md', '5: Tests') }}
