# Docker para Symfony (PHP7-FPM - NGINX - MySQL - ELK)

Docker para Symfony tiene todo lo que necesitas para desarrollar una applicacon. Este Stack corre con  [docker-compose (1.7 o superior)](https://docs.docker.com/compose/).

## Instalacion

1. Crear un archivo `.env` copiando el archivo de ejemplo `.env.dist`. Adaptar las variables de acuerdo a los requerimientos de la aplicacion.

    ```bash
    cp .env.dist .env
    ```


2. Build/run

    ```bash
    $ docker-compose build
    $ docker-compose up -d
    ```

3. Actualizar el archivo de hosts local (agregar symfony.dev)

    ```bash
    # UNIX only: get containers IP address and update host (replace IP according to your configuration)
    $ docker network inspect bridge | grep Gateway

    # unix only (on Windows, edit C:\Windows\System32\drivers\etc\hosts)
    $ sudo echo "171.17.0.1 symfony.dev" >> /etc/hosts
    ```

    **Note:** para **OS X**, fijarese  [aqui](https://docs.docker.com/docker-for-mac/networking/) y para **Windows** leer [aqui](https://docs.docker.com/docker-for-windows/#/step-4-explore-the-application-and-run-examples) (4to paso).

4. Preparar Symfony
    1. Actualizar app/config/parameters.yml

        ```yml
        # path/to/your/symfony-project/app/config/parameters.yml
        parameters:
            database_host: db
        ```

    2. Composer install & create database

        ```bash
        $ docker-compose exec php bash
        $ composer install
        # Symfony2
        $ sf doctrine:database:create
        $ sf doctrine:schema:update --force
        $ sf doctrine:fixtures:load --no-interaction
        # Symfony3
        $ sf3 doctrine:database:create
        $ sf3 doctrine:schema:update --force
        $ sf3 doctrine:fixtures:load --no-interaction
        ```

5. Disfruta :-)

## Uso

Solo ejecuta `docker-compose up -d`, y despues:

* Symfony app: visitar [symfony.dev](http://symfony.dev)  
* Symfony dev mode: visitar [symfony.dev/app_dev.php](http://symfony.dev/app_dev.php)  
* Logs (Kibana): [symfony.dev:81](http://symfony.dev:81)
* Logs (ubicacion de los archivos): logs/nginx and logs/symfony


## Funcionamiento

Mirando el archivo `docker-compose.yml` , tendremos los pasos que utiliza  `docker-compose` para crear imagenes:

* `db`: MySQL container,
* `php`: PHP-FPM en donde se monta el volumen de la aplicacion,
* `nginx`: Nginx webserver,
* `elk`: ELK stack container que usa Logstash para obtener los logs y enviarlos a  Elasticsearch para visualizarlos con Kibana.

El resultado final son los siguientes contenedores:

```bash
$ docker-compose ps
           Name                          Command               State              Ports            
--------------------------------------------------------------------------------------------------
dockersymfony_db_1            /entrypoint.sh mysqld            Up      0.0.0.0:3306->3306/tcp      
dockersymfony_elk_1           /usr/bin/supervisord -n -c ...   Up      0.0.0.0:81->80/tcp          
dockersymfony_nginx_1         nginx                            Up      443/tcp, 0.0.0.0:80->80/tcp
dockersymfony_php_1           php-fpm                          Up      0.0.0.0:9000->9000/tcp      
```

## Comandos utiles

```bash
# bash commands
$ docker-compose exec php bash

# Composer (e.g. composer update)
$ docker-compose exec php composer update

# SF commands (Tips: there is an alias inside php container)
$ docker-compose exec php php /var/www/symfony/app/console cache:clear # Symfony2
$ docker-compose exec php php /var/www/symfony/bin/console cache:clear # Symfony3
# Same command by using alias
$ docker-compose exec php bash
$ sf cache:clear

# Retrieve an IP Address (here for the nginx container)
$ docker inspect --format '{{ .NetworkSettings.Networks.dockersymfony_default.IPAddress }}' $(docker ps -f name=nginx -q)
$ docker inspect $(docker ps -f name=nginx -q) | grep IPAddress

# MySQL commands
$ docker-compose exec db mysql -uroot -p"root"

# F***ing cache/logs folder
$ sudo chmod -R 777 app/cache app/logs # Symfony2
$ sudo chmod -R 777 var/cache var/logs # Symfony3

# Check CPU consumption
$ docker stats $(docker inspect -f "{{ .Name }}" $(docker ps -q))

# Delete all containers
$ docker rm $(docker ps -aq)

# Delete all images
$ docker rmi $(docker images -q)
```

## FAQ

* Si tienes el siguiente error: `ERROR: Couldn't connect to Docker daemon at http+docker://localunixsocket - is it running?
If it's at a non-standard location, specify the URL with the DOCKER_HOST environment variable.` ?  
Ejecutar`docker-compose up -d`.

* Problemas con los permisos? Ver [este documento](http://symfony.com/doc/current/book/installation.html#checking-symfony-application-configuration-and-setup).

* Como configurar Xdebug?
Xdebug ya viene preconfigurado!
Solo hay que conectar el IDE al puerto `9001` con id key `PHPSTORM`