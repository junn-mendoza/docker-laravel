## docker-desktop on windows 10 using WSL 2

Docker Setup on windows 10 using Linux sub-system. Already installed ubuntu 20. Installing (Laravel, Mysql, Nginx) 

### Working Directory 

c:\repository

cd c:\repository

## Step 1 — Downloading Laravel and Installing Dependencies

git clone https://github.com/laravel/laravel.git laravel-app

- Now we will work on linux terminal using wsl 2 in windows

ubuntu

cd /mnt/c/repository/laravel-app

docker run --rm -v $(pwd):/app composer install

cd ..

sudo chown -R $USER:$USER laravel-app

## Step 2 — Creating the Docker Compose File

```html
version: '3'
services:

    #PHP Service
    php-fpm:
        build:
            context: .
            dockerfile: Dockerfile
        container_name: laravel-app    
        restart: unless-stopped
        tty: true
        working_dir: /var/www
        volumes:
            - ./:/var/www
            - ./docker-files/php/local.ini:/usr/local/etc/php/conf.d/local.ini
        networks:
            - app-network

    #Nginx Service
    webserver:
        image: nginx:alpine
        container_name: webserver
        restart: unless-stopped
        tty: true
        ports:
            - "8081:8081"
            - "8443:8443"
        volumes:
            - ./:/var/www
            - ./docker-files/nginx/conf.d/:/etc/nginx/conf.d/
        networks:
            - app-network

    #MySQL Service
    db:
        image: mysql:5.7.22
        container_name: db
        restart: unless-stopped
        tty: true
        ports:
            - "3307:3307"
        environment:
            MYSQL_DATABASE: laravel-app
            MYSQL_ROOT_PASSWORD: Imog3n@305
        volumes:
            - dbdata:/var/lib/mysql
            - ./docker-files/mysql/my.cnf:/etc/mysql/my.cnf
        networks:
            - app-network

#Docker Networks
networks:
    app-network:
        driver: bridge
#Volumes
volumes:
    dbdata:
        driver: local

```

## Step 4 — Creating the Dockerfile
```html
FROM php:7.2-fpm

# Copy composer.lock and composer.json
COPY composer.lock composer.json /var/www/

# Set working directory
WORKDIR /var/www

# Install dependencies
RUN apt-get update && apt-get install -y \
    build-essential \
    libpng-dev \
    libjpeg62-turbo-dev \
    libfreetype6-dev \
    locales \
    zip \
    jpegoptim optipng pngquant gifsicle \
    vim \
    unzip \
    git \
    curl

# Clear cache
RUN apt-get clean && rm -rf /var/lib/apt/lists/*

# Install extensions
RUN docker-php-ext-install pdo_mysql mbstring zip exif pcntl
RUN docker-php-ext-configure gd --with-gd --with-freetype-dir=/usr/include/ --with-jpeg-dir=/usr/include/ --with-png-dir=/usr/include/
RUN docker-php-ext-install gd

# Install composer
RUN curl -sS https://getcomposer.org/installer | php -- --install-dir=/usr/local/bin --filename=composer

# Add user for laravel application
RUN groupadd -g 1000 www
RUN useradd -u 1000 -ms /bin/bash -g www www

# Copy existing application directory contents
COPY . /var/www

# Copy existing application directory permissions
COPY --chown=www:www . /var/www

# Change current user to www
USER www

# Expose port 9000 and start php-fpm server
EXPOSE 9000
CMD ["php-fpm"]

```

## Step 5 — Configuring PHP


```html
sudo mkdir ~/laravel-app/docker-files/php
```
Next, open the local.ini file:
```html
sudo nano ~/laravel-app/php/local.ini
```

```html
upload_max_filesize=40M
post_max_size=40M
```

## Step 6 — Configuring Nginx

```html
sudo mkdir -p ~/laravel-app/docker-files/nginx/conf.d
```
Next, open the app.conf file:
```html
sudo nano nano ~/laravel-app/docker-files/nginx/conf.d/app.conf
```

```html
server {
    listen 8081;
    index index.php index.html;
    error_log  /var/log/nginx/error.log;
    access_log /var/log/nginx/access.log;
    root /var/www/public;
    location ~ \.php$ {
        try_files $uri =404;
        fastcgi_split_path_info ^(.+\.php)(/.+)$;
        fastcgi_pass laravel-app:9000;
        fastcgi_index index.php;
        include fastcgi_params;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        fastcgi_param PATH_INFO $fastcgi_path_info;
    }
    location / {
        try_files $uri $uri/ /index.php?$query_string;
        gzip_static on;
    }
}
```

## Step 7 — Configuring MySQL

```html
sudo mkdir ~/laravel-app/docker-files/mysql
```
Next, open the my.cnf file:
```html
sudo nano ~/laravel-app/docker-files/mysql/my.cnf
```
```html
[mysqld]
general_log = 1
general_log_file = /var/lib/mysql/general.log
```

## Step 8 — Modifying Environment Settings and Running the Containers

```html
sudo cp .env.example .env
```

```html
DB_CONNECTION=mysql
DB_HOST=db
DB_PORT=3306
DB_DATABASE=laravel
DB_USERNAME=laraveluser
DB_PASSWORD=your_laravel_db_password
```

Save your changes and exit your editor.
```html
docker-compose up -d
```

```html
docker ps
```

```html
docker-compose exec [image-name] php artisan key:generate

next...

docker-compose exec [image-name] php artisan config:cache
```

## Step 9 — Creating a User for MySQL

```html
docker-compose exec db bash

next...

mysql -u root -p

next...

mysql> show databases;

next...

mysql> GRANT ALL ON laravel.* TO 'laraveluser'@'%' IDENTIFIED BY 'your_laravel_db_password';

next...

mysql> FLUSH PRIVILEGES;

next...

mysql> EXIT;
```

