## docker-desktop on windows 10 using WSL 2

Docker Setup on windows 10 using Linux sub-system. Already installed ubuntu 20. Installing (Laravel, Mysql, Nginx) 

### Working Directory 

c:\repository

cd c:\repository

## Install Laravel

git clone https://github.com/laravel/laravel.git laravel-app

- Now we will work on linux terminal using wsl 2 in windows

ubuntu

cd /mnt/c/repository/laravel-app

docker run --rm -v $(pwd):/app composer install

cd ..

sudo chown -R $USER:$USER laravel-app

