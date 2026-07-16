# Setup WSL Ubuntu 24.04 sebagai Local Production Server

> Stack:
>
> -   Windows 11 + WSL2
> -   Ubuntu 24.04.4 LTS
> -   Nginx
> -   PHP 8.3 + PHP-FPM
> -   MariaDB
> -   Composer
> -   Node.js v24.14.1 (NVM)
> -   Supervisor
> -   Laravel + Vue (Vite Production Build)

## 1. Install WSL

Buka PowerShell (Administrator):

``` powershell
wsl --install
```

Restart Windows.

Install Ubuntu:

``` powershell
wsl --list --online
wsl --install Ubuntu-24.04
```

Cek powershell:

``` powershell
wsl -l -v
```

Update C:\Users\jtech\.wslconfig:

``` powershell
[wsl2]
networkingMode=mirrored
localhostForwarding=true
```
Simpan lalu jalankan:

``` powershell
wsl --shutdown
```
Cek di wsl:

``` powershell
ss -tulpn | grep 8000
```
Jika benar outputnya seperti ini:

``` powershell
tcp   LISTEN 0      4096          0.0.0.0:8000       0.0.0.0:*    users:(("php8.3",pid=3117,fd=6))
```
------------------------------------------------------------------------

## 2. Update Ubuntu

``` bash
sudo apt update
sudo apt upgrade -y
sudo apt autoremove -y
sudo timedatectl set-timezone Asia/Jakarta
```

------------------------------------------------------------------------

## 3. Install Package Dasar

``` bash
sudo apt install -y \
build-essential curl wget git zip unzip \
software-properties-common ca-certificates \
gnupg lsb-release
```

------------------------------------------------------------------------

## 4. Install PHP 8.3

``` bash
sudo add-apt-repository ppa:ondrej/php -y
sudo apt update

sudo apt install -y \
php8.3 php8.3-cli php8.3-fpm php8.3-common \
php8.3-mysql php8.3-curl php8.3-gd php8.3-intl \
php8.3-mbstring php8.3-bcmath php8.3-xml \
php8.3-zip php8.3-soap php8.3-readline php8.3-opcache
```

Cek:

``` bash
php -v
```

------------------------------------------------------------------------

## 5. Install Composer

``` bash
php -r "copy('https://getcomposer.org/installer','composer-setup.php');"
php composer-setup.php
sudo mv composer.phar /usr/local/bin/composer

composer --version
```

------------------------------------------------------------------------

## 6. Install NVM & Node.js

``` bash
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.40.3/install.sh | bash
source ~/.bashrc

nvm install 24.14.1
nvm use 24.14.1
nvm alias default 24.14.1

node -v
npm -v
```

------------------------------------------------------------------------

## 7. Install MariaDB

``` bash
sudo apt install mariadb-server -y
sudo service mariadb start
sudo mysql_secure_installation
```

Login:

``` bash
sudo mariadb
```

``` sql
CREATE USER 'jpos'@'localhost' IDENTIFIED BY 'admin123';

GRANT ALL PRIVILEGES
ON *.*
TO 'jpos'@'localhost';

FLUSH PRIVILEGES;
```

------------------------------------------------------------------------

## 8. Install Nginx

``` bash
sudo apt install nginx -y
sudo service nginx start
```

------------------------------------------------------------------------

## 9. Install Supervisor

``` bash
sudo apt install supervisor -y
```

------------------------------------------------------------------------

## 10. Clone Project

``` bash
mkdir -p ~/projects
cd ~/projects

git clone <repository> jpos
cd jpos
```

------------------------------------------------------------------------

## 11. Install Dependency

``` bash
composer install
npm install
```

------------------------------------------------------------------------

## 12. Build Frontend

``` bash
npm run build
```

------------------------------------------------------------------------

## 13. Konfigurasi Laravel

``` bash
cp .env.example .env

php artisan key:generate
```

Buat database:

``` sql
CREATE DATABASE jpos;
```

Edit `.env`:

``` env
DB_DATABASE=jpos
DB_USERNAME=jpos
DB_PASSWORD=admin123
```

Jalankan:

``` bash
php artisan migrate --seed
```

------------------------------------------------------------------------

## 14. Permission

``` bash
sudo chown -R www-data:www-data storage bootstrap/cache
sudo chmod -R 775 storage bootstrap/cache
```

------------------------------------------------------------------------

## 15. PHP-FPM

``` bash
sudo service php8.3-fpm start
```

------------------------------------------------------------------------

## 16. Konfigurasi Nginx

`/etc/nginx/sites-available/jpos`

``` nginx
server {
    listen 80;
    server_name jpos.test;

    root /home/enjat/projects/jpos/public;
    index index.php;

    location / {
        try_files $uri $uri/ /index.php?$query_string;
    }

    location ~ \.php$ {
        include snippets/fastcgi-php.conf;
        fastcgi_pass unix:/run/php/php8.3-fpm.sock;
    }

    location ~ /\. {
        deny all;
    }
}
```

Aktifkan:

``` bash
sudo ln -s /etc/nginx/sites-available/jpos /etc/nginx/sites-enabled/
sudo rm /etc/nginx/sites-enabled/default
sudo nginx -t
sudo systemctl restart nginx
```
