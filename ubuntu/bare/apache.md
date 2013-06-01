**Warning, this file assumes a secure SSH connection and puts default configuration on everything**

```bash
sudo apt-get update
sudo apt-get install git
git config --global user.name "**USER_NAME**"
git config --global user.email **your@email.com**

sudo apt-get install apache2
sudo apt-get install mysql-server libapache2-mod-auth-mysql php5-mysql
sudo mysql_install_db
sudo /usr/bin/mysql_secure_installation

sudo apt-get install php5 libapache2-mod-php5 php5-mcrypt

curl -sS https://getcomposer.org/installer | php
sudo mv composer.phar /usr/local/bin/composer

cd /var/www/
sudo chmod -R 777 /var/www/
composer create-project laravel/laravel
cd laravel
composer install
sudo chmod -R 777 app/storage
sudo wget https://raw.github.com/rtablada/laravel-vps-guide/master/ubuntu/files/apache-default /etc/apache2/sites-available/default
sudo service apache2 restart
