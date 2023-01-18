# WSL2 with PHP, Composer & NodesJS ENV for Kirby CMS with headless CMS

Install WSL2 on Windows 11 in powershell `wsl --install -d ubuntu` und run `sudo apt update && sudo apt upgrade -y` to update the system.

## Install fish shell

`sudo apt-add-repository ppa:fish-shell/release-3 && sudo apt update && sudo apt install fish -y`

show full path in terminal `set -U fish_prompt_pwd_dir_length 0`

set fish as default shell `chsh -s $(which fish)`

## Install Apache2

Install Apache 2 `sudo apt-get install apt-transport-https ca-certificates apache2 -y`

### Enable mods

`sudo a2enmod headers && sudo a2enmod rewrite && sudo a2enmod ssl && sudo a2enmod remoteip`

Add user to www-data group `sudo usermod -a -G www-data USERNAME`

## Install PHP

### Add PHP8 Repo

`sudo add-apt-repository ppa:ondrej/php -y`

### Install PHP

`sudo apt install php8.1-fpm php8.1-intl php8.1-xml php8.1-zip php8.1-mbstring php8.1-curl php8.1-ctype php8.1-gd memcached -y`

### Install and setup php-fpm

`sudo apt install libapache2-mod-fcgid && sudo a2enmod proxy_fcgi setenvif && sudo a2enconf php8.1-fpm`

### restart services

`sudo service apache2 restart && sudo service php8.1-fpm start && sudo service php8.1-fpm status`

If you go to `http://localhost` you should see the default apache2 page.

## Apache 2 virtual hosts configuration

Create a self signed certificate
`sudo openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout /etc/ssl/private/wsl.local.key -out /etc/ssl/certs/wsl.local.crt`

Create a new virtual host file `sudo nano /etc/apache2/sites-available/website.local.conf`

    <VirtualHost *:443>
        ServerName website.local

        DocumentRoot /var/www/website/public/
        <Directory /var/www/website/public>
            AllowOverride all
            Require all granted
        </Directory>

        ErrorLog /var/log/apache2/website-error.log
        CustomLog /var/log/apache2/website-access.log combined

        SSLEngine On
        SSLCertificateFile      /etc/ssl/certs/wsl.local.crt
        SSLCertificateKeyFile   /etc/ssl/private/wsl.local.key
        SSLProtocol All -SSLv2 -SSLv3
    </VirtualHost>

Create Directory and a sample content `sudo mkdir /var/www/website/ && sudo mkdir /var/www/website/public && echo "<?php phpinfo();?>" | sudo tee /var/www/website/public/index.php`

### Enable the site

`sudo a2ensite website.local.conf && sudo service apache2 reload`

### Edit Windows .hosts file

    # WSL
    ::1	    website.local
    127.0.0.1   website.local

You can find your website here: https://website.local/

## Install Composer

`sudo apt install composer`

### NVM for fish shell

Install Fisher Plugin Manager `curl https://git.io/fisher --create-dirs -sLo ~/.config/fish/functions/fisher.fish`

`fisher install jorgebucaran/nvm.fish`
`nvm install --lts`

load LTS in fish startup `echo "nvm use lts" >> ~/.config/fish/config.fish`

see options at: https://github.com/jorgebucaran/nvm.fish

## Git config globally

`git config --global user.name "FIRST_NAME LAST_NAME"`
`git config --global user.email "MY_NAME@example.com"`
