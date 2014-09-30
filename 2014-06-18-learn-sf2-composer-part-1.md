---
layout: post
title: Aprende Symfony2 - parte 1: Composer
tags:
    - Symfony2
    - técnico
    - serie Aprende Symfony2
---

¿No sabes nada del framework [Symfony2](http://symfony.com/), y te gustaría
leer una guía rápida para aprender cómo utilizarlo, y cómo funciona?

Entonces este artículo es para ti :) .

No me malinterpretes, tarde o temprano tendrás que leer la
[documentación](http://symfony.com/doc/current/index.html), y deberás practicar
mucho si quieres dominarlo. Pero por ahora esta guía debería ser un buen
comienzo.

En el primer artículo de esta serie, conocerás
[Composer](https://getcomposer.org/), que te ayudará con la instalación y
actualización de librerías de terceros.

## Creando el proyecto

Para entender mejor cómo funciona Symfony2, no usaremos la
[distribución estándar de Symfony](http://symfony.com/distributions), sino que
empezaremos desde cero con los archivos estrictamente necesarios.

Creemos nuestro proyecto:

    mkdir knight
    cd knight
    git init

## Instalando Composer

Cuando desarrollas un proyecto, lo último que quieres es perder el tiempo
reinventando la rueda, así que normalmente utilizas librerías de terceros.
Estas librerías tienen su propio ciclo de desarrollo: pueden reparar bugs y
lanzar nuevas funcionalidades una vez las has instalado, así que necesitarás
actualizarlas de vez en cuando.

[Composer](https://getcomposer.org/) facilita esta labor de tal manera que
nunca más tendrás que preocuparte por las nuevas versiones. Primero,
descárgalo:

    curl -sS https://getcomposer.org/installer | php
    sudo mv composer.phar /usr/local/bin/composer

¡Ya está! Si todos los proyectos fueran tan fáciles de instalar...

## Instalando y actualizando Symfony2

En realidad, Symfony2 es sólo un nombre bajo el que se agrupan muchas librerías
que podrían ser usadas individualmente (incluso puedes usarlas en otros
frameworks, CMS o proyectos, como han hecho
[Drupal](http://symfony.com/projects/drupal),
[phpBB](http://symfony.com/projects/phpbb),
[Laravel](http://symfony.com/projects/laravel),
[eZ Publish](http://symfony.com/projects/ezpublish),
[OroCRM](http://symfony.com/projects/orocrm) o
[Piwik](http://symfony.com/projects/piwik)).

*Nota*: Las librerías de Symfony2 se llaman `componentes`.

Composer se creó para instalar librerías, así que vamos a usarlo:

    composer require 'symfony/symfony:~2.5' # instalamos todas las librerías de sf2

Este comando realizará las siguientes tareas:

1. Crear un archivo de configuración llamado `composer.json` si no existe ya
2. Añadir `symfony/symfony: ~2.5` a ese archivo (útil para futuros
`composer install`)
3. Descargar efectivamente Symfony en el directorio `vendor/symfony/symfony`
4. Crear un archivo `composer.lock`

Más tarde, para actualizar estas dependencias, bastará con ejecutar `composer
update`.

*Nota*: Las librerías de las que depende tu proyecto se llaman `dependencias`.

El comando recorrerá el archivo `composer.lock` para saber qué versión ha sido
instalada (p.e. 2.5.0) y comprobará si hay una versión más reciente disponible.
Para más información sobre la manera en que Composer maneja las versiones, ver
[este artículo de Igor](https://igor.io/2013/01/07/composer-versioning.html).

Esto significa que puedes ignorar el directorio `vendor`:

    echo '/vendor/*' >> .gitignore

Si un miembro de tu equipo quiere instalar el proyecto, sólo necesitará clonar el tu
repositorio y ejecutar `composer install`, que realiza las siguientes tareas:

1. Lee el archivo `composer.json` para recoger la lista de dependencias
2. Lee el archivo `composer.lock` comprobar la versión instalada por el
*commiter*
3. Descarga las dependencias con la versión especificada en el archivo lock
(aunque haya una nueva versión disponible)

Si una dependencia aparece en el archivo `composer.json` pero no en el
`composer.lock`, Composer descargará la versión coincidente más reciente que
esté disponible y la añadirá al lock.

¡Esto significa que todo el mundo tendrá la misma versión instalada! Si sólo
permites a una persona ejecutar `composer update`, está garantizado.

## Autloading

Gracias a que Composer sabe dónde está cada clase de cada librería instalada,
ofrece una magnífica funcionalidad:
[autoloading](http://www.php.net/manual/en/language.oop5.autoload.php).

Para resumir, cada vez que una clase es instanciada, Composer incluye
automáticamente el archivo en el que fue declarada.

Tu propio código también puede beneficiarse de ello. Necesitas editar el
archivo `composer.json`:

    {
        "require": {
            "symfony/symfony": "~2.5"
        },
        "autoload": {
            "psr-4": {
                "": "src/"
            }
        }
    }

Y ejecutar el siguiente comando para que se contemplen los cambios:

    composer update

Esta configuración le dice a Composer que vamos a seguir el estándar
[PSR-4](http://www.php-fig.org/psr/psr-4/) y que vamos a poner nuestro código
en el directorio `src`.

*Nota*: PSR requiere que tu código siga algunas convenciones:

* Crea una clase por cada archivo
* Dale el mismo nombre a tu archivo y tu clase
* Usa la ruta de la clase como namespace

Por ejemplo: el archivo
`src/Knight/ApplicationBundle/KnightApplicationBundle.php`
contiene una clase llamada `KnightApplicationBundle` localizada en el namespace
`Knight\ApplicationBundle`.

No te preocupes demasiado por esto ahora.

## Conclusión

...Y esto es todo lo que necesitas saber sobre Composer por ahora. Hagamos un
commit de nuestro trabajo:

    git add -A
    git commit -m 'Symfony2 instalado'

Espero que te haya servido, ¡permanece atento a los próximos artículos!

### Próximos artículos

* {{ link('posts/2014-06-25-learn-sf2-empty-app-part-2.md', '2: Aplicación vacía') }}
* {{ link('posts/2014-07-02-learn-sf2-bundles-part-3.md', '3: Bundles') }}
* {{ link('posts/2014-07-12-learn-sf2-controllers-part-4.md', '4: Controladores') }}
* {{ link('posts/2014-07-20-learn-sf2-tests-part-5.md', '5: Tests') }}
* {{ link('posts/2014-07-23-learn-sf2-conclusion.md', 'Conclusión') }}
