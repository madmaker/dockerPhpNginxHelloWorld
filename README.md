# Docker + Php + Nginx HelloWorld

## Create a project with following folders:
*Создайте проект со следующими папками*

/projectRootDir\
-> /docker\
-> /src\
In "src" folder I will create index.php file that executes phpinfo().\
*В папке src я создам файл index.php, который выведет на экран информацию о php с помощью функции phpinfo()*


## Create nginx config site.conf in "docker" folder:
*Создайте файл конфигурации nginx в папке docker: site.conf*

~~~apacheconf
server {
    server_name myapp.loc;

    root /var/www/myapp;
    index index.php index.html index.htm;

    access_log /var/log/nginx/php-access.log;
    error_log /var/log/nginx/php-error.log;

    location / {
        try_files $uri $uri/ /index.php?$query_string;
    }

    # PHP-FPM Configuration Nginx
    location ~ \.php$ {
        try_files $uri = 404;
        fastcgi_split_path_info ^(.+\.php)(/.+)$;
        fastcgi_pass php:9000;
        fastcgi_index index.php;
        include fastcgi_params;
        fastcgi_param REQUEST_URI $request_uri;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        fastcgi_param PATH_INFO $fastcgi_path_info;
    }
}
~~~

`fastcgi_pass php:9000;` - this is what tells nginx how to connect to php container\
*Это то, что сообщает nginx, как подключаться к контейнеру php*

Other strings are same to default nginx config\
*Остальные строки ничем не отличаются от обычного конфига nginx*

## Edit /etc/hosts file on host machine, and add a record:
*Отредактируйте файл hosts (/etc/hosts для unix, system32/drivers/etc/hosts для windows) и добавьте туда следующую запись:*

`127.0.0.1       myapp.loc`

## Create docker-compose.yml:
*Создайте файл docker-compose.yml*

~~~yml
version: "3.7"
services:
  web:
    image: nginx:1.17
    ports:
      - 80:80
    volumes:
      - ./src:/var/www/myapp
      - ./docker/site.conf:/etc/nginx/conf.d/site.conf
    depends_on:
      - php
  php:
    image: php:8-fpm
    volumes:
      - ./src:/var/www/myapp
      # - ./docker/php.ini:/usr/local/etc/php/php.ini
~~~

Here we run two containers: web - nginx server and php - php8.\
*Мы запускаем два контейнера: web - сервер nginx и php - php8 *

## Create index.php file in src folder:
*Создайте файл index.php в папке src*

~~~php
<?php
phpinfo();
~~~

## Run Docker:
*Запустите docker*
`docker-compose up`

## Open http://myapp.loc in browser
*Откройте http://myapp.loc в браузере*
