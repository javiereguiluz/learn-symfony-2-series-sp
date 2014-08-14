---
layout: post
title: Aprende Symfony2 - parte 5: Tests
tags:
    - Symfony2
    - técnico
    - serie Aprende Symfony2
---

Este es el quinto artículo de la serie para aprender sobre
[el framework Symfony2](http://symfony.com/).
Echa un vistazo a los cuatro primeros:

1. {{ link('posts/2014-06-18-learn-sf2-composer-part-1.md', 'Composer') }}
2. {{ link('posts/2014-06-25-learn-sf2-empty-app-part-2.md', 'Aplicación Vacía') }}
3. {{ link('posts/2014-07-02-learn-sf2-bundles-part-3.md', 'Bundles') }}
4. {{ link('posts/2014-07-12-learn-sf2-controllers-part-4.md', 'Controladores') }}

En los anteriores artículos comenzamos a crear una aplicación vacía con los
siguientes archivos:

    .
    ├── app
    │   ├── AppKernel.php
    │   ├── cache
    │   │   └── .gitkeep
    │   ├── config
    │   │   ├── config.yml
    │   │   └── routing.yml
    │   └── logs
    │       └── .gitkeep
    ├── composer.json
    ├── composer.lock
    ├── src
    │   └── Knight
    │       └── ApplicationBundle
    │           ├── Controller
    │           │   └── ApiController.php
    │           └── KnightApplicationBundle.php
    ├── .gitignore
    └── web
        └── app.php

Ejecutar `composer install` debería crear el directorio `vendor`, que hemos
ignorado en git.

Aquí está el [el repositorio en el que encontrarás el código hasta ahora](https://github.com/gnugat/learning-symfony2/tree/4-controllers).

En este artículo crearemos tests funcionales utilizando PHPUnit.

## Instalando PHPUnit

[PHPUnit](http://phpunit.de/) es un popular framework de testing.
Su nombre engaña: puedes escribir todo tipo de tests con él (unitarios, pero
  también funcionales, *end to end*, etc).

Instalémoslo en nuestro proyecto:

    composer require --dev "phpunit/phpunit:~4.1"

La opción `--dev` evitará que Composer instale PHPUnit cuando ejecutemos
`composer install --no-dev`: se usa en producción (para reducir descargas).

Necesitaremos crear un archivo de configuración que le diga a PHPUnit que
ejecute los tests situados en `src/Knight/ApplicationBundle/Tests`, y que use
Composer como autoloader:

    <?xml version="1.0" encoding="UTF-8"?>
    <!-- File: app/phpunit.xml.dist -->

    <!-- http://phpunit.de/manual/current/en/appendixes.configuration.html -->
    <phpunit
        backupGlobals="false"
        colors="true"
        syntaxCheck="false"
        bootstrap="../vendor/autoload.php">

        <testsuites>
            <testsuite name="Functional Test Suite">
                <directory>../src/Knight/ApplicationBundle/Tests</directory>
            </testsuite>
        </testsuites>

    </phpunit>

*Nota*: [Por convención](http://symfony.com/doc/current/cookbook/bundles/best_practices.html#directory-structure)
deberías poner tus tests en `src/Knight/ApplicationBundle/Tests`. No es
obligatorio, pero si quieres que otra gente encuentre las cosas donde esperan
encontrarlas, es mejor que las sigas ;) .

Este archivo lleva el sufijo `.dist` porque pretendemos que cada desarrollador
pueda sobreescribir la configuración creando un archivo `app/phpunit.xml`. Sólo
el archivo distribuído debe ser publicado:

    echo '/app/phpunit.xml' >> .gitignore
    git add -A
    git commit -m 'PHPUnit instalado'

## Entornos

Para nuestros tests funcionales, utilizaremos la clase `WebTestCase`: instancia
nuestro `AppKernel` con el entorno `test`. También usa el servicio
`test.client`, que está desactivado por defecto.

Para actiarlo, debemos modificar la configuración:

    # File: app/config/config.yml
    framework:
        secret: "Three can keep a secret, if two of them are dead."
        router:
            resource: %kernel.root_dir%/config/routing.yml

        # test: ~

Normalmente no queremos que nuestra configuración sea la misma en nuestros tests
y en nuestro servidor de producción. Para eso están los entornos. Coloquemos la
configuración específica de nuestros tests en un archivo diferente:

    # File: app/config/config_test.yml
    imports:
        - { resource: config.yml }

    framework:
        test: ~

*Nota*: el parámetro `imports` te permite incluír otros archivos de
configuración.
Así, puedes sobreescribir los parámetros incluídos, o añadir nuevos.

Deberíamos cambiar el método `registerContainerConfiguration` de la clase
`AppKernel` para que cargue la configuración de los tests dependiendo del
entorno:

    <?php
    // File: app/AppKernel.php

    use Symfony\Component\HttpKernel\Kernel;
    use Symfony\Component\Config\Loader\LoaderInterface;

    class AppKernel extends Kernel
    {
        public function registerBundles()
        {
            return array(
                new Symfony\Bundle\FrameworkBundle\FrameworkBundle(),
                new Knight\ApplicationBundle\KnightApplicationBundle(),
            );
        }

        public function registerContainerConfiguration(LoaderInterface $loader)
        {
            $file = 'config';
            if ('test' === $this->getEnvironment()) {
                $file .= '_test';
            }
            $loader->load(__DIR__."/config/$file.yml");
        }
    }

Subamos nuestro trabajo al repositorio:

    git add -A
    git commit -m 'Añadida  la configuración de testing'

## Tests funcionales

Nuestro test debe comprobar que la aplicación se comporta como esperamos. No
comprobaremos que realmente cumple todas nuestras expectativas de negocio. Esto
significa que comprobar el código de estado HTTP es más que suficiente.

Creemos el directorio:

    mkdir -p src/Knight/ApplicationBundle/Tests/Controller

*Nota*: De nuevp [por convención](http://symfony.com/doc/current/book/testing.html#unit-tests),
tu estructura de directorios de test debe replicar la de tu bundle.

Y nuestro primer test funcional:

    <?php
    // File: src/Knight/ApplicationBundle/Tests/Controller/ApiControllerTest.php

    namespace Knight/ApplicationBundle/Tests/Controller;

    use Symfony\Bundle\FrameworkBundle\Test\WebTestCase;

    class ApiControllerTest extends WebTestCase
    {
        public function testOfferingTheRightThing()
        {
            $method = 'POST';
            $uri = '/api/ni';
            $parameters = array();
            $files = array();
            $server = array();
            $content = json_encode(array(
                'offering' => 'shrubbery',
            ));

            $client = static::createClient();
            $client->request($method, $uri, $parameters, $files, $server, $content);
            $response = $client->getResponse();

            $this->assertTrue($response->isSuccessful());
        }
    }

Para comprobar que pasamos el test, ejecuta el siguiente comando:

    ./vendor/bin/phpunit -c app

Composer ha instalado un archivo binario en `vendor/bin`, y la opción `-c` te
permite decírle a PHPUnit dónde está la configuración (en `./app`).

Parece que nuestro test es un poco largo por culpa de los parámetros... Podemos
mejorar esto con métodos auxiliares:

    <?php
    // File: src/Knight/ApplicationBundle/Tests/Controller/ApiControllerTest.php

    namespace Knight/ApplicationBundle/Tests/Controller;

    use Symfony\Bundle\FrameworkBundle\Test\WebTestCase;

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

            $this->assertTrue($response->isSuccessful());
        }
    }

Comprueba que aún pasamos el test:

    ./vendor/bin/phpunit -c app

El método `isSuccessful` de Response sólo comprueba que el código de estado es
200.

Aquí está el test para el caso de error:

    <?php
    // File: src/Knight/ApplicationBundle/Tests/Controller/ApiControllerTest.php

    namespace Knight/ApplicationBundle/Tests/Controller;

    use Symfony\Bundle\FrameworkBundle\Test\WebTestCase;

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

            $this->assertTrue($response->isSuccessful());
        }

        public function testOfferingTheWrongThing()
        {
            $response = $this->post('/api/ni', array('offering' => 'hareng'));

            $this->assertFalse($response->isSuccessful());
        }
    }

Ejecuta los tests:

    ./vendor/bin/phpunit -c app

*Nota*: A partir de aquí, debería convertirse en habitual ejecutar los tests.
Asegúrate de ejecutarlos siempre que termines un cambio, y de ejecutarlos de
nuevo antes de añadir nada al repositorio.

## Tests funcionales de la API Rest

En mi humilde opinión, comprobar que el código de estado es 200 y no comprobar
el contenido de la respuesta es más que suficiente para un test funcional.

Al crear una API REST, puede ser útil testear más precisamente el código de
estatus. Nuestra aplicación es una API REST, así que hagámoslo:

    <?php
    // File: src/Knight/ApplicationBundle/Tests/Controller/ApiControllerTest.php

    namespace Knight/ApplicationBundle/Tests/Controller;

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

Ejecuta los tests:

    ./vendor/bin/phpunit -c app

¡Verde! ¡Es suficiente recompensa como para subir nuestro trabajo al repositorio
y dar la jornada por terminada!

    git add -A
    git commit -m 'Tests añadidos'

## Conclusión

¡Ejecutar `./vendor/bin/phpunit -c app` es más sencillo que tener que ejecutar
manualmente HTTPie (como en el artículo anterior)!

Escribir tests funcionales es fácil y rápido, lo único que debes hacer es
comprobar si el código de status HTTP es correcto (y para una API REST,
  comprobar con precisión el código de status HTTP).

El próximo artículo será un resumen de esta serie, ¡espero que te haya gustado!

### Próximos artículos

* {{ link('posts/2014-07-23-learn-sf2-conclusion.md', 'Conclusión') }}

### Artículos anteriores

* {{ link('posts/2014-06-18-learn-sf2-composer-part-1.md', '1: Composer') }}
* {{ link('posts/2014-06-25-learn-sf2-empty-app-part-2.md', '2: Aplicación Vacía') }}
* {{ link('posts/2014-07-02-learn-sf2-bundles-part-3.md', '3: Bundles') }}
* {{ link('posts/2014-07-12-learn-sf2-controllers-part-4.md', '4: Controladores') }}
