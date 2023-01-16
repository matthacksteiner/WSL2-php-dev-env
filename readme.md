# Install WSL PHP ENV
- Install Apache 2 `sudo apt-get install apt-transport-https ca-certificates apache2`
- Enable mods
  sudo a2enmod headers
  sudo a2enmod rewrite
  sudo a2enmod ssl
  sudo a2enmod remoteip
- Add user to www-data group `sudo usermod -a -G www-data`
- Add PHP8 Repo `sudo add-apt-repository ppa:ondrej/php`
- Install PHP/webserver `sudo apt install -y php8.2-fpm php8.2-mbstring php8.2-curl php8.2-ctype php8.2-gd
