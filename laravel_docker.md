# Laravel con Docker

## Crear la carpeta para alojar los proyectos

```
mkdir laravel; cd laravel
```

> Si ya tenemos un repositorio clonado, abriremos un terminal en esa carpeta.

> En Windows, usar CMD.EXE para lanzar los comandos, no PowerShell.

## Descargar Laradock

1. Clonar el repositorio:

    ```
    git clone https://github.com/Laradock/laradock.git
    ```

    o añadirlo como submódulo, si ya tenemos un proyecto Git:

    ```
    git submodule add https://github.com/Laradock/laradock.git
    ```

    > Para que funcione, tiene que estar instalado [el cliente de línea de comandos de Git](https://git-scm.com/downloads).

2. Copiar el fichero `env-example` a `.env`:

    Linux y macOS

    ```
    cd laradock && cp env-example .env
    ```

    Windows

    ```
    cd laradock; copy env-example .env
    ```

3. Editar el fichero `.env` de la carpeta `laradock`:

    - Modificar el driver de base de datos de phpMyAdmin: `PMA_DB_ENGINE=mariadb`
    - Si disponemos de más de una instalación de Laradock, modificar la variable `COMPOSE_PROJECT_NAME` y asignarle un nombre único para que los contenedores tengan nombres diferentes.

## Nuevo proyecto de Laravel

(Opcional) Reemplazar `app` por el nombre del proyecto a partir de aquí.

### Generar el proyecto

Linux y macOS

```
docker run -it --rm --name php-cli \
    -v composer_cache:/home/docker/.composer/cache \
    -v $PWD:/usr/src/app thecodingmachine/php:7.3-v2-slim-cli \
    composer create-project --prefer-dist laravel/laravel app
```

Windows

```
docker run -it --rm --name php-cli ^
    -v composer_cache:/home/docker/.composer/cache ^
    -v %CD%:/usr/src/app thecodingmachine/php:7.3-v2-slim-cli ^
    composer create-project --prefer-dist laravel/laravel app
```

### (Opcional) Poner el proyecto bajo control de versiones

> No lanzar este comando si se ha instalado Laradock como submódulo.

El siguiente comando crea un repositorio nuevo de Git en la carpeta del proyecto. 

```
cd app; git init; git add .; git commit -m "Initial commit"; cd ..
```

### Configurar un nuevo sitio en Laradock 

1. Ir a `laradock/nginx/sites` y duplicar `laravel.conf.example` a `app.conf`.

2. Modificar en el fichero `app.conf` estas dos líneas, cambiado `laravel` por el nombre del proyecto:

    ```
    server_name app.test;
    root /var/www/app/public;
    ```

### Editar fichero de hosts

Añadir a `/etc/hosts` (en Windows `C:\Windows\System32\Drivers\etc\hosts`):

```
127.0.0.1	app.test
```

### (Re)arrancar los contenedores

Los comandos de `docker-compose` se lanzan en la carpeta `laradock`.

Arrancar los contenedores necesarios:

```
docker-compose up -d nginx mariadb phpmyadmin workspace
```

Y para reiniciar un contenedor concreto:

```
docker-compose restart nginx
```

### Crear la base de datos

1. Acceder a [phpMyAdmin](http://localhost:8080)

    - Servidor `mariadb` y usuario `root/root`.
    - Crear la base de datos `app` y el usuario `app/app`.

2. Editar el .env de la aplicación

    ```
    DB_CONNECTION=mysql
    DB_HOST=mariadb
    DB_PORT=3306
    DB_DATABASE=app
    DB_USERNAME=app
    DB_PASSWORD=app
    ```
    
#### Parche para Windows

La base de datos no arranca correctamente debido al sistema de ficheros en que corre Docker. Para solucionarlo, editar el fichero `laradock/docker-compose.yml` y modificar:

1. En la sección `volumes`, alrededor de la línea 24, añadir un volumen nuevo para alojar los datos de mariadb:

    ```yml
      mariadb_data:
        driver: ${VOLUMES_DRIVER}
    ```

2. En la sección correspondiente a `mariadb`, alrededor de la línea 390, editar la sección `volumes` y reemplazar:

    ```yml
    ### MariaDB ##############################################
        ...
          volumes:
            - ${DATA_PATH_HOST}/mariadb:/var/lib/mysql
        ...
    ```

    Por:

    ```yml
    ### MariaDB ##############################################
        ...
          volumes:
            - mariadb_data:/var/lib/mysql
        ...
    ```

3. Reiniciar los contenedores.

### Acceder al sitio web

Página principal: http://app.test

## Utilidades

### Lanzar comandos en el proyecto (composer, artisan, npm...)

```
docker-compose exec workspace /bin/bash

cd app
```

Y después el comando que necesitemos. Por ejemplo:

```
php artisan tinker
```

o

```
php artisan make:model Tarea -mcr
```

### Consola de MariaDB

```
docker-compose exec mariadb mysql -u root -proot
```

### Sitio predeterminado

El sitio por defecto de nginx en [localhost](http://localhost) muestra el directorio `laravel/public`. Se puede dejar ahí un fichero `index.html` para que no de un error 404.

### Añadir soporte para fechas en castellano

Editar el fichero `.env` de laradock y activar la opción `PHP_FPM_INSTALL_ADDITIONAL_LOCALES=true`. 

En la variable `PHP_FPM_ADDITIONAL_LOCALES` escribir la lista de idiomas adicionales, como por ejemplo `es_ES.UTF-8` para castellano.

