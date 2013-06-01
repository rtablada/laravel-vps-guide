# Laravel 4 with Apache on Ubuntu 12.10

To get started with this guide, I assume you have a running installation of Ubuntu with terminal access (either through SSH to a remote server/VPS or on a terminal prompt in a virtual/physical machine).

## Contents

- [Initial Setup](#initial-setup)
- [Install Git](#install-git)
- [Setup ZSH Prompt](#setup-zsh-optional)
- [Install LEMP](#install-the-lemp-stack)
- [Install Composer](#installing-composer-globally)
- [Install Laravel](#installing-laravel-4)
- [Setup Nginx for Laravel](#setting-up-apache-for-laravel-4)
- [Remote Git Repositories](#pulling-remote-git-repositories)

## Initial Setup

This will guide you through setting up your users and making your SSH access a bit more secure.

Login to your shell if you are using a remote server you will need to login via SSH `ssh root@**IP_ADDRESS**`.

Next you should change your root password to something secure.
`passwd` Type in your new password at the prompt.

Now we should create a custom user to make things a bit more secure (doing everything as the root user is dangerous). `adduser **USER_NAME**`

Now we need to give your new user some permissions as an administrator. `visudo` Then find the line that looks like `root	ALL=(ALL:ALL) ALL`. After this line add a line with `**USER_NAME**	ALL=(ALL:ALL) ALL`

Finally, we will modify some of the properties to allow this new user to login via SSH, and we will change some settings to dissuade hacking. `nano /etc/ssh/sshd_config`

Find the line that has `Port 22` change this number to any valid port number you would like, this will add some security as now we won't be using the default port, making it harder to hack in.

Find the line that starts with `Protocol ` and make sure that it is set to `Protocol 2`. This is the SSH protocol version and allows for more secure and stable connections.

Find the line that lists `PermitRootLogin yes`. Change this to `PermitRootLogin no`. This makes it so that hackers cannot get direct root access without knowing your newly created user credentials.

Find the line that lists `UseDNS yes`. Change this to `UseDNS no` Though this provides minimal security, this stops hackers from being able to use your domain name to attack your server. This also means that you will have to SSH in using the IP address of your server.

At the end of this file add a line with `AllowUsers **USER_NAME**` This allows your newly created user to login via SSH.

Save this file. To wrap things up, you will need to restart SSH `reload ssh`.

Login to your server using your new credentials.

## Install Git

This is a quick command `sudo apt-get update && sudo apt-get install git`. You will also want to set your name and email for your git config: `git config --global user.name "**USER_NAME**"` & `git config --global user.email **your@email.com***`

## Setup ZSH (Optional)

Having a boring console isn't very fun. ZSH allows for better tab completion than standard Bash shell included in Ubuntu. The best part of ZSH is that it is a subset of Bash (meaning it will run any standard commands Bash will).

Install ZSH `sudo apt-get install zsh`.

Next you will want to install Oh-My-ZSH which gives you some good off of the shelf configuration for ZSH. `curl -L https://github.com/robbyrussell/oh-my-zsh/raw/master/tools/install.sh | sh`

Now you have to actually make Oh My ZSH your configuration framework. `cp ~/.oh-my-zsh/templates/zshrc.zsh-template ~/.zshrc`.

You can modify the theme used by changing the ZSH_THEME in your .zshrc file. `nano ~/.zshrc`. I like a theme called 'doubleend' from [Andrew Burgess](https://github.com/andrew8088/oh-my-zsh/blob/master/themes/doubleend.zsh-theme). You will need to download this as it isn't included in the standard Oh My ZSH themes. `cd ~/.oh-my-zsh/themes` then `wget https://raw.github.com/andrew8088/oh-my-zsh/master/themes/doubleend.zsh-theme`.

Last, we will need to change your default shell prompt from Bash to ZSH. `chsh -s /bin/zsh`. And to get all the awesomeness of ZSH you will have to re-login.

## Install the LEMP Stack

A LEMP stack is a light-weight solution compared to the weight of Apache. For this stack we will be installing Nginx (pronounced engine-x), MySQL, and PHP FPM (a web optimized light-weight build of PHP).

Let's start by installing Nginx. `sudo apt-get install nginx`

Now we'll tackle the 'M' - MySQL. If you don't already know, MySQL is an Open Source Database that will store the majority of dynamic info for your application. This script is a bit longer because it also installs necessary modules to make MySQL talk to Apache and PHP. This will ask for a root user password: set it to something secure, this user will have access to all of your databases (including the database that controls user authentication for MySQL). `sudo apt-get install mysql-server`.

To finish off our LEMP stack, we have to get PHP FPM up and running. `sudo apt-get install php5-fpm`

Even though PHP FPM gives performance enhancements for our page rendering, it does not handle command line scripts very well without configuration that would be out of the scope of this guide. So we are going to install regular PHP alongside of PHP FPM. `sudo apt-get install php5 php5-mcrypt`

## Installing Composer Globally

For Laravel 4, you're going to need Composer. Don't worry, it is super easy to install! `curl -sS https://getcomposer.org/installer | php` & `sudo mv composer.phar /usr/local/bin/composer`

Now when you run the command `composer` you should get a composer options prompt.

## Installing Laravel 4

With Laravel 4 being stable you can now install it in a single command, which is awesome. But first we will have to go to the right directory. When you installed your LAMP stack, it created a folder for your websites. `cd /var/www/`

You will also have to set the permissions for this folder `sudo chmod -R 777 /var/www/` This will allow you to read and write to the websites folder and it will let Apache read from here.

Now we can pull down Laravel 4 from composer `composer create-project laravel/laravel`.

We will move into the Laravel installation now and install the framework dependencies `cd laravel` & `composer install`.

Finally, you need to set the permissions for Laravel to do all of its caching magic: `sudo chmod -R 777 app/storage`

## Setting Up Nginx for Laravel 4

Now we can configure Nginx to serve up our application. We will need to dig into the default site. This is based on the configuration from [Dayle Rees](https://github.com/daylerees/laravel-website-configs/blob/master/nginx.conf) `sudo nano /etc/nginx/sites-available/default`

- server - this contains all of the rules for a single service. This is similar to VirtualHost block in Apache.
	- `listen 80` - this makes Nginx catch all website requests
	- `server localhost` - this tells Nginx what to listen to. You can set this to your Fully Qualified Domain or your public IP Address.
	- `root /var/www/laravel;` - This is where our project is located
	- `index index.php;` - What should we load when we get a raw domain or IP?
	- `location /` - this block replicates the .htaccess that ships with Laravel and removes index.php from the application request
		- `try_files   $uri $uri/ /index.php?$query_string;`
	- ```
			if (!-d $request_filename) {
        		rewrite     ^/(.+)/$ /$1 permanent;
    		}
    	```


## Pulling Remote Git Repositories

Having a bare Laravel install is great, but doing all of your editing through SSH on a VPS is not realistic. Instead, let's wire this up to pull from an existing project you have setup on github or another git repo!

Move into the websites directory: `cd /var/www`
Remove the old Laravel 4 install `rm -rf laravel`
Pull in your repository `git pull **url_to_repo** laravel`
Composer install components `cd laravel` & `composer install`
Make git ignore file permissions (saves tons of headaches of detached HEAD problems) - git config core.filemode false
Set file permissions `sudo chmod -R 777 app/storage`

That's it. Now when you want to pull down changes from your repository just run `cd /var/www/laravel` & `git pull` (if you change any composer packages you will also need to run `composer update`).
