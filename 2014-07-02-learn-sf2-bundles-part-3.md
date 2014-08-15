---
layout: post
title: Aprende Symfony2 - parte 3: Bundles
tags:
    - Symfony2
    - técnico
    - serie Aprende Symfony2
---

Este es el tercer artículo de la serie para aprender sobre
[el framework Symfony2](http://symfony.com/).
Echa un vistazo a los dos primeros:

* {{ link('posts/2014-06-18-learn-sf2-composer-part-1.md', '1: Composer') }}
* {{ link('posts/2014-06-25-learn-sf2-empty-app-part-2.md', '2: Aplicación vacía') }}

En los anteriores artículos empezamos creando nuestro proyecto vacío con los
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
    ├── .gitignore
    └── web
        └── app.php

Ejecutar `composer install` debería crear el directorio `vendor`, que hemos
ignorado en git.

Aquí está el [el repositorio en el que encontrarás el código hasta ahora](https://github.com/gnugat/learning-symfony2/releases/tag/2-empty-application).

Vamos a ver qué es un bundle.

## Creando el bundle de la aplicación

Necesitaremos un caso de uso para que nuestros trozos de código tengan sentido.
Así que ahí va: ¡los Caballeros Que Dicen 'Ni' necesitan un webservice! Deberá
decir 'ni' si el usuario no complace sus exigencias. Para hacerlo, el usuario
deberá postear ¡una almáciga! (shruberry).

Vamos a definir el primer bundle de nuestra aplicación, para tener así un lugar
en el que poner nuestro código. Para ello necesitamos crear su directorio:

    mkdir -p src/Knight/ApplicationBundle

Después, la clase, que extiende `Symfony\Component\HttpKernel\Bundle\Bundle`:

    <?php
    // File: src/Knight/ApplicationBundle/KnightApplicationBundle.php

    namespace Knight\ApplicationBundle;

    use Symfony\Component\HttpKernel\Bundle\Bundle;

    class KnightApplicationBundle extends Bundle
    {
    }

Finalmente registramos el bundle en nuestra aplicación:

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
                new Knight\ApplicationBundle\KnightApplicationBundle(), // <-- Aquí!
            );
        }

        public function registerContainerConfiguration(LoaderInterface $loader)
        {
            $loader->load(__DIR__.'/config/config.yml');
        }
    }

Hagamos un commit al repositorio:

    git add -A
    git commit -m 'Creado el bundle de la aplicación'

## Los Bundles te permiten extender el DIC de la aplicación

La clase `KnightApplicationBundle` extiende la siguiente:

    <?php

    namespace Symfony\Component\HttpKernel\Bundle;

    use Symfony\Component\DependencyInjection\ContainerAware;
    use Symfony\Component\Console\Application;

    abstract class Bundle extends ContainerAware implements BundleInterface
    {
        public function getContainerExtension();
        public function registerCommands(Application $application);
    }

*Nota*: Sólo se muestra la parte que nos interesa.

Estos dos métodos hacen que el bundle sea capaz de *autodescubrir* sus comandos
y su extensión del Contenedor de Inyección de Dependencias (DIC), si se
utiliza la siguiente estructura de directorios:

    .
    ├── Command
    │   └── *Command.php
    ├── DependencyInjection
    │   └── KnightApplicationExtension.php
    └── KnightApplicationBundle.php

*Nota*: el único archivo requerido es el`KnightApplicationBundle.php`.

El nombre de un bundle (en nuestro ejemplo `KnightApplication`) se compone de:

* el nombre del *vendor* (el de nuestro cliente: `Knight`)
* el nombre real del bundle (`Application`)

Por tu bien, elige una palabra corta como nombre del vendor y del bundle (no es
  una condición requerida, pero te lo aconsejo).

La clase `KnightApplicationExtension` te permite manipular el DIC (normalmente
  cargarás un archivo de configuración localizado en
  `Resources/config/services.xml`).

Y precisamente ese es el propósito de los bundles: registrar servicios en el
DIC de la aplicación.

### Nota al margen sobre el DIC y los Servicios

Los Servicios y la Inyección de Dependencias escapan del ámbito de esta
serie. De todos modos, si quieres descubrir de qué va eso, echa un vistazo a
estos dos artículos:

* {{ link('posts/2014-01-22-ioc-di-and-service-locator.md', 'Inversion of Control, Dependency Injection, Dependency Injection Container and Service Locator') }}
* {{ link('posts/2014-01-29-sf2-di-component-by-example.md', 'Symfony2 Dependency Injection component, by example') }}

*Nota*: esto nos indica la verdadera naturaleza de los componentes de Symfony2.
Son librerías de terceros que pueden usarse por separado fuera del framework.

### Nota al margen sobre comandos

El componente de consola de Symfony2 te permite crear aplicaciones de línea de
comandos (CLI applications). Una aplicación puede tener uno o varios comandos.
Para aprender más sobre comandos, echa un vistazo a este artículo:

* {{ link('posts/2014-04-09-sf2-console-component-by-example.md', 'Symfony2 Console component, by example') }}

*Nota*: los comandos rebasan el ámbito de este artículo,
pero es conveniente mencionarlos.

## Dos tipos de bundle

Hay dos tipos de bundle:

* de integración como librerías de terceros (reusables, se comparten entre
  aplicaciones)
* de aplicación (no reusables y dedicados a tu modelo de negocio)

Miremos a la [librería snappy de KnpLabs](https://github.com/KnpLabs/snappy): te
permite generar un PDF a partir de una página HTML y puede ser usado en
cualquier aplicación (aplicaciones no-symfony, incluso aplicaciones sin
  framework).

La clase que permite esta generación es
`Knp\Bundle\SnappyBundle\Snappy\LoggableGenerator`: su construcción es un poco
cansina. Para arreglarlo, podemos definir su construcción en el DIC, y
afortunadamente ya hay un bundle que hace esto por nosotros:
[KnpSnappyBundle](https://github.com/KnpLabs/KnpSnappyBundle).

Ahí tenemos un buen ejemplo de los bundles del primer tipo.

Sobre el segundo tipo: necesitaremos integrar nuestro propio código en nuestra
aplicación Symfony2 un día u otro. Podríamos elegir el largo y tortuoso camino
de escribir un montón de inicializaciones y configuraciones, ¡o podríamos usar
un bundle que haga el trabajo por nosotros automáticamente!

A veces, encontraremos aplicaciones que tienen muchos bundles, para así poder
categorizar las funcionalidades en módulos. Esto no es necesario, y es un poco
cansino, si se me permite: podemos simplemente crear carpetas en un único bundle
para categorizar nuestros módulos.

La creación de varios bundles nencesita algunos pasos manuales adicionales.
Además no tiene mucho sentido, ya que un bundle es, teóricamente, una unidad
desacoplada: si creamos un UserBundle, un FrontendBundle, un BlogBundle y un
ForumBundle, nos encontraremos con bundles dependientes unos de otros, a menudo
con dependencias cíclicas, y perderemos el tiempo decidiendo dónde poner nuevas
clases (que puede que necesiten a los otros tres bundles).

Mi consejo: crea un único bundle para tu aplicación. Si más tarde descubres que
has creado un conjunto de clases que tendría sentido reusar en otros proyectos
(sean proyectos Symfony2 o no), entonces quizá puedas extraerlas para crear una
librería de terceros. Y después podrás finalmente crear un bundle para integrar
esa librería en aplicaciones Symfony2.

## Conclusión

Los bundles son una manera de extender el Contenedor de Inyección de
Dependencias: son la capa de pegamento que une tu código y las aplicaciones
Symfony2.

Siguen convenciones que no han sido *hard coded* (puedes sobreescribir cualqiuer
  cosa), permitiendo *autodescubrir* algunas clases muy convenientes.

Gracias por leer. En el próximo artículo, ¡crearemos controladores!

### Próximos artículos

* {{ link('posts/2014-07-12-learn-sf2-controllers-part-4.md', '4: Controladores') }}
* {{ link('posts/2014-07-20-learn-sf2-tests-part-5.md', '5: Tests') }}
* {{ link('posts/2014-07-23-learn-sf2-conclusion.md', 'Conclusión') }}

### Artículos anteriores

* {{ link('posts/2014-06-18-learn-sf2-composer-part-1.md', '1: Composer') }}
* {{ link('posts/2014-06-25-learn-sf2-empty-app-part-2.md', '2: Aplicación Vacía') }}

### Recursos

Aquí os dejo un buen artículo sobre cómo crear bundles reusables:

* [Use only infrastructural bundles in Symfony2, by Elnur Abdurrakhimov](http://elnur.pro/use-only-infrastructural-bundles-in-symfony/)

¿No te gusta seguir convenciones, y estás preparado para escribir un montón de
código de inicialización y configuración? Aquí tienes (no te lo recomiendo):

* [Should everything really be a bundle in Symfony2?](http://stackoverflow.com/questions/9999433/should-everything-really-be-a-bundle-in-symfony-2-x/10001019#10001019)
* [Yes, you can have low coupling in a Symfony2 application](http://danielribeiro.org/blog/yes-you-can-have-low-coupling-in-a-symfony-standard-edition-application/)
* [Symfony2 without bundles, by Elnur Abdurrakhimov, by Daniel Ribeiro](http://elnur.pro/symfony-without-bundles/)
* [Symfony2 some things I dont like about bundles, by Matthias Noback](http://php-and-symfony.matthiasnoback.nl/2013/10/symfony2-some-things-i-dont-like-about-bundles/)
* [Symfony2 console commands as services why, by Matthias Noback](http://php-and-symfony.matthiasnoback.nl/2013/10/symfony2-console-commands-as-services-why/)
* [Naked bundles, slides by Matthias Noback](http://www.slideshare.net/matthiasnoback/high-quality-symfony-bundles-tutorial-dutch-php-conference-2014)

Incluyo estos links únicamente porque me gusta cómo explican el funcionamiento
de Symfony2 por detrás, pero no aplicaría lo que dicen en una aplicación real,
ya que incluye una complejidad que no compensa (de todos modos, esta es mi
  humilde opinión).
