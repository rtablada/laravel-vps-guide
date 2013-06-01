# Laravel 4 with Apache on Ubuntu 12.10

To get started with this guide, I assume you have a running installation of Ubuntu with terminal access (either through SSH to a remote server/VPS or on a terminal prompt in a virtual/physical machine).

## Contents

- [Initial Setup](#initial-setup)
- [Install Git](#install-git)
- [Setup ZSH Prompt](#setup-zsh-optional)
- [Install LAMP](#install-the-lamp-stack)

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

## Install the LAMP Stack

Now we get to the fun part: installing the LAMP (Linux Apache MySQL PHP) components.

First we install the Apache HTTP Server. This will connect HTTP requests to the proper services (in our case, PHP). `sudo apt-get install apache2`.

Now we'll tackle the 'M' - MySQL. If you don't already know, MySQL is an Open Source Database that will store the majority of dynamic info for your application. This script is a bit longer because it also installs necessary modules to make MySQL talk to Apache and PHP. This will ask for a root user password: set it to something secure, this user will have access to all of your databases (including the database that controls user authentication for MySQL). `sudo apt-get install mysql-server libapache2-mod-auth-mysql php5-mysql`

Now we have to finish setting up MySQL. `sudo mysql_install_db` & `sudo /usr/bin/mysql_secure_installation` Answer 'y' to all of the questions. This will set some default settings that make your MySQL install a bit more secure (like disabling remote root login).

Finally, this is what you all been waitin' for 'aint it? What people pay paper for... Now we're going to install PHP. This isn't your grandad's (or your shared host's) PHP 5.2 or anything either, at the time of this guide, this will install PHP 5.4.6 (if you are on Ubuntu 12.04, you'll get 5.3.x which isn't very fun)! And this will handle installing the PHP MCrypt module which is used in Laravel's Hash. `sudo apt-get install php5 libapache2-mod-php5 php5-mcrypt`

## Installing Composer Globally

For Laravel 4, you're going to need Composer. Don't worry, it is super easy to install! `curl -sS https://getcomposer.org/installer | php` & `sudo mv composer.phar /usr/local/bin/composer`

Now when you run the command `composer` you should get a composer options prompt.

## Installing Laravel 4

With Laravel 4 being stable you can now install it in a single command, which is awesome. But first we will have to go to the right directory. When you installed your LAMP stack, it created a folder for your websites. `cd /var/www/`

You will also have to set the permissions for this folder `sudo chmod -R 777 /var/www/` This will allow you to read and write to the websites folder and it will let Apache read from here.

Now we can pull down Laravel 4 from composer `composer create-project laravel/laravel`.

We will move into the Laravel installation now and install the framework dependencies `cd laravel` & `composer install`.

Finally, you need to set the permissions for Laravel to do all of its caching magic: `sudo chmod -R 777 app/storage`

## Setting up apache for Laravel

By default, Apache reads from `/var/www` which isn't what we want. So we will have to modify our Apache configuration.

First we have to let Apache know to look for PHP files. `sudo nano /etc/apache2/mods-enabled/dir.conf`

In this file, find the line that starts with DirectoryIndex and add `index.php` so it will look something like this `DirectoryIndex index.php index.html index.cgi index.pl index.php index.xhtml index.htm` **Note:** The order of these matter, by placing index.php first, we prioritize it over other index files that may exist. Now save the file.

Now we will modify our default site configuration in Apache. This gets into some more complicated configuration options so I won't be able to explain everything, but I will get you up and running. `sudo nano /etc/apache2/sites-available/default` I suggest using ZSH or Bash tab completion to make sure that you are editing the proper file (sometimes it isn't always named 'default').

Now let's go setting by setting to get everything working.

- VirtualHost - Directs Apache what it needs to listen to *:80 catches all HTTP requests. Every option in this tag will apply to requests that match this pattern and port.
	- ServerAdmin - list your email
	- DocumentRoot - `/var/www/laravel/public` (This is where we installed Laravel)
	- Directory - This will be a XML-ish block and you should make it look like `<Directory /var/www/laravel/public>`
		- `Options Indexes FollowSymLinks MultiViews`
			- Indexes - Makes Apache look for index.php (and all of the various extensions we set up earlier)
			- FollowSymLinks - Allows Unix style symbolic links to be used to put content on your site
			- MultiViews - I think this allows Laravel to capture the URIs. I'm not too clear on what it does, but you need this option enabled.
		- `AllowOverride None` - .htaccess files will not be allowed to override configuration options, just add options and modules.
		- `Order allow,deny` - Makes the server accept requests
		- `allow from all` - Makes the server accept requests from any client

The rest of this file should be left to its defaults so the final product looks a bit like [this](files/apache-default).
