# WSL2 with PHP, Composer & NodesJS ENV

##### Original sources:

https://www.karlomikus.com/blog/php-development-environment-with-wsl-2-and-debian

https://www.digitalocean.com/community/tutorials/how-to-create-a-self-signed-ssl-certificate-for-apache-in-ubuntu-22-04

## Install Ubuntu from Powershell

Install WSL2 on Windows 11 in powershell `wsl --install -d ubuntu` und run `sudo apt update && sudo apt upgrade -y` to update the system.

## Install fish shell

`sudo apt-add-repository ppa:fish-shell/release-3 && sudo apt update && sudo apt install fish -y`

show full path in terminal `set -U fish_prompt_pwd_dir_length 0`

set fish as default shell `chsh -s $(which fish)`

## Install Apache2

Install Apache 2 `sudo apt-get install apt-transport-https ca-certificates apache2 -y`

### Enable mods

`sudo a2enmod headers && sudo a2enmod rewrite && sudo a2enmod ssl && sudo a2enmod remoteip`

Fix permission for current User
`sudo chown -R USER:USER /var/www`

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

Create a new virtual host file `sudo nano /etc/apache2/sites-available/website.local.conf`

    <VirtualHost *:80>
        ServerName website.local
        DocumentRoot /var/www/website.local/public/
        <Directory /var/www/website.local/public>
            Options Indexes FollowSymLinks
            AllowOverride All
            Require all granted
        </Directory>
        ErrorLog /var/log/apache2/error.log
        CustomLog /var/log/apache2/access.log combined
    </VirtualHost>

Create Directory and a sample content `sudo mkdir /var/www/website/ && sudo mkdir /var/www/website/public && echo "<?php phpinfo();?>" | sudo tee /var/www/website/public/index.php`

### Enable the site

`sudo a2ensite website.local.conf && sudo service apache2 reload`

### Fix all permission on /var/www/

`sudo chmod -R 777 /var/www/`

### Check Apache config

`sudo apache2ctl -S`

### Edit Windows .hosts file

    # WSL
    ::1	    website.local
    127.0.0.1   website.local

You can find your website here: https://website.local/

## Install Composer

Download Composer with curl
`cd ~ && curl -sS https://getcomposer.org/installer -o /tmp/composer-setup.php`

Install Composer
`sudo php /tmp/composer-setup.php --install-dir=/usr/local/bin --filename=composer`

### NVM for fish shell

Install Fisher Plugin Manager
`curl https://git.io/fisher --create-dirs -sLo ~/.config/fish/functions/fisher.fish`

Install NVM
`fisher install jorgebucaran/nvm.fish`

Install latest LTS Version
`nvm install lts`

load LTS in fish startup `echo "nvm use lts" >> ~/.config/fish/config.fish`

see options at: https://github.com/jorgebucaran/nvm.fish

## Git config globally

`git config --global user.name "FIRST_NAME LAST_NAME" && git config --global user.email "MY_NAME@example.com"`

## Git ignore file permissions globally
`git config --global core.fileMode false`

## Create a SSH key

`ssh-keygen -t rsa`

## Install Fira Code font

`sudo apt install fonts-firacode`

## bash file for Apache

    #!/bin/bash

    # Check if the script is being run in sudo mode
    if [ "$EUID" -ne 0 ]; then
        echo "This script must be run with sudo privileges."
        sudo "$0" "$@"
        exit
    fi

    # Prompt the user for server name
    read -p "Enter server name (e.g. example.com): " server_name

    # Set document root to /var/www
    doc_root="/var/www/${server_name}"

    # Generate Apache 2 config
    config_file="/etc/apache2/sites-available/${server_name}.conf"
    echo "<VirtualHost *:80>" >$config_file
    echo "    ServerName $server_name" >>$config_file
    echo "    DocumentRoot $doc_root/public/" >>$config_file
    echo "    <Directory $doc_root/public>" >>$config_file
    echo "        Options Indexes FollowSymLinks" >>$config_file
    echo "        AllowOverride All" >>$config_file
    echo "        Require all granted" >>$config_file
    echo "    </Directory>" >>$config_file
    echo "    ErrorLog /var/log/apache2/error.log" >>$config_file
    echo "    CustomLog /var/log/apache2/access.log combined" >>$config_file
    echo "</VirtualHost>" >>$config_file

    # Create index.html file
    sudo mkdir $doc_root
    sudo mkdir $doc_root/public
    echo "<h1>It works!</h1>" | sudo tee $doc_root/public/index.html
    
    # fix permissions
    sudo chmod -R 777 /var/www/ *

    
    # Linux only: Add the server name to the beginning of /etc/hosts
    sudo sed -i "1i127.0.0.1 ${server_name}" /etc/hosts

    # Enable the new site and restart Apache 2 service
    sudo a2ensite $server_name
    sudo service apache2 reload

    echo "Apache 2 configuration for $server_name has been created."
