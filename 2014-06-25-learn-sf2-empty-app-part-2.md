---
layout: post
title: Aprende Symfony2 - parte 2: Aplicación vacía
tags:
    - Symfony2
    - técnico
    - serie Aprende Symfony2
---

Este es el segundo artículo de la sere para aprender sobre
[el framework Symfony2](http://symfony.com/).
Échale un vistazo al primero:
{{ link('posts/2014-06-18-learn-sf2-composer-part-1.md', 'Composer') }}.

En el primer artículo empezamos a crear nuestro proyecto vacío con los
siguientes archivos:

    .
    ├── composer.json
    ├── composer.lock
    └── .gitignore

Ejecutar `composer install` debería crear el directorio `vendor`, que hemos
ignorado en git.

Aquí está el [el repositorio en el que encontrarás el código hasta ahora](https://github.com/gnugat/learning-symfony2/tree/1-composer).

Veamos cómo crear una aplicación Symfony2 vacía.

## El controlador frontal

Empecemos por el principio, vamos a crear un archivo index que actuará como
controlador frontal: será el único punto de entrada a nuestra aplicación y
decidirá qué página mostrar.

Creemos su directorio:

    mkdir web

Después, el archivo:

    <?php
    // File: web/app.php

    use Symfony\Component\HttpFoundation\Request;

    require_once __DIR__.'/../vendor/autoload.php';
    require_once __DIR__.'/../app/AppKernel.php';

    $kernel = new AppKernel('prod', false);
    $request = Request::createFromGlobals();
    $response = $kernel->handle($request);
    $response->send();
    $kernel->terminate($request, $response);

Primero incluímos el autoloader de Composer: requerirá todos los archivos
necesarios.

Después creamos una instancia de nuestro Kernel, en el entorno de producción
y con las herramientas de debug deshabilitadas. Esta clase actúa como un
servidor web: recibe una HTTP request y devuelve una HTTP response.

`Request::createFromGlobals()` crea una representación de la request HTTP.
Se rellena con las variables super globales de PHP (`$_GET`, `$_POST`, etc).

Entonces el kernel se hace cargo de la request. Para no dar demasiadas
explicaciones, diremos que lo que hará será encontrar el controlador asociado
a la url solicitada. Es responsabilidad del controlador frontal devolver una
representación de la response HTTP (ver
`Symfony\Component\HttpFoundation\Response`).

El método `$response->send()` simplemente llama a la función `header` de PHP, e
imprime una cadena de texto que será el body de la response (habitualmente HTML,
 JSON o lo que tú quieras).

Finalmente, el método `$kernel->terminate()` llamará a cualquier tarea suscrita
al evento `kernel.terminate` event. Esto te permite devolver una respuesta tan
pronto como sea posible, y después ejecutar algunas acciones más, como enviar
emails.

*Nota*: los eventos se escapan al propósito de este artículo, pero es
conveniente mencionarlos.

## Creando el kernel de la aplicación

[El componente HttpKernel](http://symfony.com/doc/current/components/http_kernel/introduction.html)
nos da la clase `Kernel`, a la que extenderemos.

Crea el siguiente directorio:

    mkdir app

Y después el archivo del kernel:

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
            );
        }

        public function registerContainerConfiguration(LoaderInterface $loader)
        {
            $loader->load(__DIR__.'/config/config.yml');
        }
    }

Esta clase cargará la configuración del proyecto. También es aquí donde
registrarás los bundles del proyecto. Hablaremos más sobre bundles en el
próximo artículo, por ahora lo único que necesitas saber es que son una especie
de plugins.

El kernel tiene la responsabilidad de buscar en cada bundle registrado para
obtener su configuración.

El bundle `FrameworkBundle` define algunos servicios que te permiten elegir
qué activar desde la configuración.

*Nota*: Los servicios son objetos que hacen una sola cosa, y la hacen bien.
Ofrecen exactamente lo que anuncian: un servicio. Aprenderemos más sobre ellos
en uno de los próximos artículos.

Necesitamos definir algunas configuraciones para poder hacerlo funcionar
debidamente.

Crea su directorio:

    mkdir app/config

Y el archivo YAML de configuración:

    # File: app/config/config.yml
    framework:
        secret: "Three can keep a secret, if two of them are dead."

El parámetro `secret` se usa como semilla para generar cadenas aleatorias (por
  ejemplo tokens CSRF).

Ahora que tenemos la estructura de nuestra aplicación, subámosla al repositorio:

    git add -A
    git commit -m 'Estructura de la aplicación creada'

### Logs y caché

Necesitarás crear también los directorios `logs` y `cache`:

    mkdir app/{cache,logs}
    touch app/{cache,logs}/.gitkeep

Git no permite subir al repositorio un directorio vacío, de ahí los archivos
`.gitkeep`.

Dado que los archivos en estos directorios serán temporales, los ignoraremos:

    echo '/app/cache/*' >> .gitignore
    echo '/app/logs/*' >> .gitignore
    git add -A
    git add -f app/cache/.gitkeep
    git add -f app/logs/.gitkeep
    git commit -m 'Directorios temporales creados'

### Configuración de Apache

Para que tu sitio sea accesible, necesitarás configurar tu servidor web. El
proceso está muy bien explicado
[en la documentación](http://symfony.com/doc/current/cookbook/configuration/web_server_configuration.html),
así que esta es la pinta de un vhost de apache:

    <VirtualHost *:80>
        ServerName knight.local

        DocumentRoot /home/loic.chardonnet/Projects/gnugat/knight/web

        ErrorLog "/home/loic.chardonnet/Projects/gnugat/knight/app/logs/apache_errors.log"
        CustomLog "/home/loic.chardonnet/Projects/gnugat/knight/app/logs/apache_accesses.log" common

        <Directory /home/loic.chardonnet/Projects/gnugat/knight/web>
            Options Indexes FollowSymLinks MultiViews
            AllowOverride None
            Order allow,deny
            allow from all
            <IfModule mod_rewrite.c>
                RewriteEngine On
                RewriteCond %{REQUEST_FILENAME} !-f
                RewriteRule ^(.*)$ /app.php [QSA,L]
            </IfModule>
        </Directory>
    </VirtualHost>

Si te encuentras con problemas de permisos (como no poder escribir en `cache`
  y `logs`), puede que quieras cambiar las variables de entorno
  `APACHE_RUN_USER` y `APACHE_RUN_GROUP`, presentes en `/etc/apache2/envvars`,
  por tu propio usuario y su grupo.

## Conclusión

Una aplicación Symfony2 sigue este patrón: un controlador frontal asocia una
URL a un controlador, que recoge una request HTTP y devuelve una response HTTP.

El siguiente artículo va sobre bundles, así que permanece atento :) .

### Próximos artículos

* {{ link('posts/2014-07-02-learn-sf2-bundles-part-3.md', '3: Bundles') }}
* {{ link('posts/2014-07-12-learn-sf2-controllers-part-4.md', '4: Controladores') }}
* {{ link('posts/2014-07-20-learn-sf2-tests-part-5.md', '5: Tests') }}
* {{ link('posts/2014-07-23-learn-sf2-conclusion.md', 'Conclusión') }}

### Artículos anteriores

* {{ link('posts/2014-06-18-learn-sf2-composer-part-1.md', '1: Composer') }}
