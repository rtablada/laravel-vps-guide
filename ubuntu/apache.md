# Laravel 4 with Apache on Ubuntu 12.04

To get started with this guide, I assume you have a running installation of Ubuntu with terminal access (either through SSH to a remote server/VPS or on a terminal prompt in a virtual/physical machine).

## Contents

- [Initial Setup](#initial-setup)
- [Install Git](#install-git)
- [Setup ZSH Prompt](#setup-zsh-optional)

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

This is a quick command `sudo apt-get update && sudo apt-get install git`. You will also want to set your name and email for your git config: `git config --global user.name "**USER_NAME**"` & `git config --global user.email **your@email.com***

## Setup ZSH (Optional)

Having a boring console isn't very fun. ZSH allows for better tab completion than standard Bash shell included in Ubuntu. The best part of ZSH is that it is a subset of Bash (meaning it will run any standard commands Bash will).

Install ZSH `sudo apt-get install zsh`.

Next you will want to install Oh-My-ZSH which gives you some good off of the shelf configuration for ZSH. `curl -L https://github.com/robbyrussell/oh-my-zsh/raw/master/tools/install.sh | sh`

Now you have to actually make Oh My ZSH your configuration framework. `cp ~/.oh-my-zsh/templates/zshrc.zsh-template ~/.zshrc`.

You can modify the theme used by changing the ZSH_THEME in your .zshrc file. `nano ~/.zshrc`. I like a theme called 'doubleend' from [Andrew Burgess](https://github.com/andrew8088/oh-my-zsh/blob/master/themes/doubleend.zsh-theme). You will need to download this as it isn't included in the standard Oh My ZSH themes. `cd ~/.oh-my-zsh/themes` then `wget https://raw.github.com/andrew8088/oh-my-zsh/master/themes/doubleend.zsh-theme`.

Last, we will need to change your default shell prompt from Bash to ZSH. `chsh -s /bin/zsh`. And to get all the awesomeness of ZSH you will have to re-login.

