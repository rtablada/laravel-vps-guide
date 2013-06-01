# Laravel 4 with Apache on Ubuntu 12.04

To get started with this guide, I assume you have a running installation of Ubuntu with terminal access (either through SSH to a remote server/VPS or on a terminal prompt in a virtual/physical machine).

## Contents

- [Initial Setup](#initial-setup)

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
