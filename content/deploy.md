---
title: "Deploying laravel over VPS"
date: 2025-01-09
description: "a notes regarding deployment of application through ssh"
tags:
  - laravel
  - php
  - vps
  - ssh
---

# Using hetzner to deploy application

## Choosing OS

Normally a linux os and the most common is Ubuntu

### Updating the OS and adding a user

after choosing the os, in ubuntu you can update the os by doing the following command:

```bash
sudo apt update # this will check for updates
apt list --upgradable # this will update the os

# when prompted for configuring openssh-server
# choose :
# Keep the local version currently installed

```

After updating you can now add the user using the following command:

```bash
adduser <username>
```

and fill up the necessary info like passwords

add the newly created user to the sudo group so that you can run sudo commands while still not having the same previlages with the root user

```bash
usermod -aG sudo <username>
```

You can now switch to the newly `sudo` access user with

```bash
su - <usernamed>
```

then test it with

```bash
sudo ls /
```

### Installing SSH

SSH is a protocol that allows you to securely log into a remote server.

- after creating a user, create a ssh key so that we can disable the password login authentication

  Instructions:

  1.  run this command (need openssh to be installed)

  ```bash
  ssh-keygen -t ed25519 -C "your_email@example.com"
  ```

  2.  Enter the file path or pre "Enter for the default" (~/.ssh/id_ed25519 - this is from github)
  3.  Set a passphrase for added security and configrm the passphrase

  View the pub key

  1.  Once completed, navigate to the .ssh directory in your user folder to locate the generated keys.
  2.  Open the .pub file with a text editor and copy the content or run this command
      (dont forget that its .pub )

  ```bash
  cat ~/.ssh/id_ed25519.pub | wl-copy
  ```

  wl-copy for wayland, otherwise xclip

  3.  then paste the ssh key content in the ssh key content with your name on it

  4.  then deploy your ssh instance

  Another Instruction using ecdsa

## Hardening ssh

In order to avoid a bruteforce ssh attack, password must be disabled

1. (in local machine)

   ```bash
   ssh-copy-id <user>@<vps-id-addr>
   ```

   youll have to authenticate through this command and with this will enable a keybased authentication

   then test it by login through ssh again and this time it should not ask for password again

   > [!IMPORTANT] custom ssh name
   >
   > by default ssh will read `id_rsa` and `id_ed25519` .
   > if you have a custom name for the ssh key, you need to specify it in the command
   > for example: `ssh-copy-id -i ~/.ssh/<your_custom_ssh> <user>@<vps-id-addr>`
   >
   > `<your_custom_ssh>` - is your PRIVATE key not public(.pub)
   >
   > or add a shortcut for ssh (see number 6)

2. editing the sshd_config

   ```bash
   sudo vim /etc/ssh/ssh_config
   ```

   here you can edit the following:

   - uncomment `PasswordAuthentication yes` and change it to `PasswordAuthentication no`
   - find `PermitRootLogin yes` to `PermitRootLogin no` (remove the ability to login as root)
   - find `UsePAM yes` to `UsePAM no`

   > [!NOTE]
   >
   > also check /etc/ssh/ssh_config.d/
   > for another file like `50-cloud-init.conf`

   if it exists also apply the necessary steps above to it (or remove it )

3. reload systemctl

   to apply the changes reload the ssh service with:

   ```bash
   sudo systemctl reload ssh
   ```

4. test
   to test you can logout the ssh and try logging in with root user `root@<vps-id-addr>`
   if you weren't able to login meaning it is successful

5. (optional) change the port to something else
   in the `/etc/ssh/ssh_config`

> [!TIP] Port
>
> you can change the port to something else to avoid the attacks from automated scanners
>
> you can do this by finding and uncommenting the `port 22` to like `port 8000`

> [!WARNING]
>
> make sure to take note of the changed port

6. (optional) Adding a shortcut to ssh

   If you have a custom ssh name for your vps creating a shortcut will make sure that ssh will check that `private` ssh key instead of the default one (`id_ed25519`)

   To create one do the following:

   - cd to your `~/.ssh/` and create a `config`
   - inside the config file is the following code:

   ```bash
   Host <ssh-alias> #foo-server
      HostName <vps-id-addr> #192.168.123.456
      User <vps-user> #bar
      IdentityFile <.ssh-private-key> #~/.ssh/id_foo
      IdentityFile <.ssh-private-key> #~/.ssh/id_foo_bar
   ```

   - `<ssh-alias>` - the alias that you want to use when you run the ssh command (e.g. `ssh foo-server`)
   - `<vps-id-addr>` - the ip address of your vps
   - `<vps-user>` - the user that you want to use to login to the vps
   - `<.ssh-private-key>` - the private key that you want to use to login to the vps

   then you can login to the vps with the following command:

   ```bash
   ssh <ssh-alias>
   ```

## Pointing DNS

Add a dns record (cloudflare) and point it to your server ip address

```bash
ip addr
```

type: A
name: @
points to: ip address
ttl : 300

### cloudflare and subdomain

if you're managing your domain with cloudflare you can use the following command to point your subdomain to your vps ip address

Cloudflare dashboard -> dns -> records -> add records

- type: A
- name: @ or <subdomain>
- IPv4 address: vps ip address
- Proxy status: Proxied(orange cloud)
- ttl : auto

<subdomain> just the name of the subdomain that you want. It will automatically become <subdomain>.domain.com

> [!TIP]
>
> install `tmux` so that when, for example, got disconnected you can just tmux attach

after pointing out it may take some time to propgate it.. you can check it if its already propagated with

```bash
nslookup <your website url>
```

once confirmed you can try to ssh using the website url instead in this format:

```bash
ssh <username>@<websiteurl>
# ssh foo@bar.sit
```

## Installing php and composer

When using ubuntu as the os first you need to add the repository for php:

(make sure that your [os is updated](#updating-the-os-and-adding-a-user))

```bash
sudo add-apt-repository ppa:ondrej/php
```

after that you can install php

```bash
sudo apt install php8.3-cli
```

Install the common php extensions

```bash
sudo apt install php8.3-common php8.3-curl php8.3-mbstring php8.3-xml php8.3-zip php8.3-gd php8.3-mysql php8.3-bcmath php8.3-sqlite3 php8.3-imagick -y
```

`-y` is to automatically answer yes to all prompts

### Installing composer

```bash
sudo apt install composer
```

then check if it is installed with `composer -V`

## Installing mysql

use this command to install mysql in the server

```bash
sudo apt install mysql-server
```

### mysql security steps

run this command for the setup

```bash
sudo mysql_secure_installation
```

1. Setting up validate password component
   This checks the strength of password

   value: `Y|y`

   - then it will prompt for password level validation policy
     value: 0 = low, 1 = med , 2= strong.
     choose: `2`
     Strong password requires: `numeric, mixed case, special characters and dictionary`

2. Remove the anonymous users
   choose: `y|Y` when prompted

3. Disallow remote root login
   choose: `y|Y` when prompted

4. Remove the test database and access to it
   choose: `y|Y` when prompted

5. Reload privilege tables now
   choose: `y|Y` when prompted

### setting up mysql

login to mysql with the following command:

```bash
sudo mysql
```

this will open a mysql shell and you can run the following commands:

1. create the database:

   ```bash
   CREATE DATABASE <database_name>;
   USE <database_name>;

   show databases; # will check all database

   ```

   <database_name> - the name of the database you want to create. dont forget the `;`

2. create the user

   ```bash
   CREATE USER '<username>'@'localhost' IDENTIFIED BY '<password>';
   ```

   `<username>` - your username to be use to connect to the database you created
   `localhos` - will use the server's local ip as the host
   `<password>` - the password for your database user.
   password should follow the strong password requirements (mention in the [mysql security steps](#mysql-security-steps))

3. Grant previlages

   ```bash

   GRANT ALL PRIVILEGES ON <database_name>.* TO '<username>'@'localhost';

   FLUSH PRIVILEGES;
   ```

   Grant previlages to the newly created user in order to have a permission to interact with the database(read write).

   Then flush privileges afterwards to apply the changes after granting the previlages. This is force apply the privileges since by default mysql caches the privileges of the database

## Simple setup for sqlite

When using sqlite as a database there are simple security measure that you can do.

- File permission and location:

```bash
# Set restrictive permissions
chmod 640 database.sqlite
# Only the web server user and owner should have access
chown www-data:www-data database.sqlite
```

Sqlite database should and must not in the /public directory. By default laravel is looking your /database.

- setup firewall to only access https,ssh

## Setting up github

For security reasons, setting up github ssh keybased authentication is a must.

The process is almost the same with [installing ssh](#installing-ssh) but with a few differences.

1. generate a new ssh key

   in your vps got to the `.ssh` directory and run the following command:

   ```bash
   ssh-keygen -t ed25519
   ```

   name the ssh key for example `id_github`

2. get a copy of the ssh key public key
   then check and copy for the content of the public key with the following command:

   for wayland:

   ```bash
   cat ~/.ssh/id_github.pub | wl-copy
   ```

   for x11:

   ```bash
   cat ~/.ssh/id_github.pub | xclip
   ```

   if both ways are not working just cat the file and manually copy it.

3. Paste the key to github

   In github account go to `settings -> ssh and gpg keys -> new ssh key`

   in the new ssh key section fill up the following:

   - title: the name that you want
   - key: the content of the **copied** public key from your vps

   then save it.

   you can checkout the [github documentation](https://docs.github.com/en/authentication/connecting-to-github-with-ssh/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent) for more information.

4. Save the ssh key to the ssh agent

   start the ssh agent with the following command:
   `eval "$(ssh-agent -s)"`

   then add the key to the ssh agent with the following command:

   ```bash
   ssh-add ~/.ssh/id_github #private path to the ssh key that you added with github
   ```

   lastly, test the connection with the following command:

   ```bash
   ssh -T git@github.com
   ```

   if you receive a success message, you are good to go and can now continue [installing the package and dependencies](#installing-the-package-and-dependencies)

## Setting up nginx

for webserver and reverse proxy nginx is a common option although traefik is also popular.

to install:

```bash
sudo apt install nginx
```

check the version if its installed with `nginx -v`
then you can copy your vps id address and pase it in your browser
if it loads to nginx welcome page, meaning it is installed

## Setting up node

you can use a version manager like nvm to easily manage your node and switch to the versions easily.

to install using curl:

```bash
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.40.1/install.sh | bash
```

and run the following to your vps

```bash
export NVM_DIR="$([ -z "${XDG_CONFIG_HOME-}" ] && printf %s "${HOME}/.nvm" || printf %s "${XDG_CONFIG_HOME}/nvm")"
[ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh" # This loads nvm
```

this will add node path in your vps but to persist it you can add it in your rc file.

- check for the current shell
  `echo $SHELL`
  normally it will be `/bin/bash`.
  then you can add the code block above in the `.bashrc` file.

install the lateast lts version of node using the following command:

```bash
nvm install --lts
```

## Cloning and setting up the project

since we are using nginx as a reverse proxy, we need to clone the project in the default directory for the nginx
which is located in `/var/www/html/`

if you cd to the directory and run `ls` you will see the following:

```bash
index.nginx-debian.html
```

this is the file that we saw when we accessed the nginx server through the vps ip address.

When cloning the project to this directory, you will encounter a permission denied error.

To fix this, you can take ownership of the directory and its contents with the following command:

```bash
sudo chown -R <username>:<username> /var/www/html
```

<username> is the username of the user of the vps.

then you can clone the project with the following command:

```bash
git clone <github-project-url>
```

this will error because github ssh is not enabled in the vps. to set this up you can refer [setting up github](#setting-up-github)

### installing the package and dependencies

after setting up github and cloning your project to html directory, you can run the following command to install the composer and its dependencies:

```bash
composer install
```

> [!NOTE]
> if this fails. most likely, the extension is not installed.
> check the error and install the indicated extension
>
> then try again.

- Generate key
  When installing a laravel project, you will need to generate a key for the project.

  Things to remember when generating a key:

  - `.env` file should be in the root directory of the project
    and with the correct permission

    You can run `sudo chown <user> .env`

  Then you can run `php artisan key:generate`

  > [!TIP]
  > dont forget to modify the .env with the credentials needed for production

- migrate the databases
  if the database is empty , especially if you are using sqlite, you might need to migrate your database

  ```bash
  php artisan migrate
  # or
  php artisan migrate --seed
  ```

- installing and building node dependencies

  ```bash
  # pnpm is not resources hungry
  npm install or pnpm install
  ```

  then run

  ```bash
  npm run build
  ```

  if encountered an error, you need to modify the tsc to be executable

  ```bash
  chmod +x node_modules/.bin/tsc
  # if using vue
  chmod +x node_modules/.bin/vue-tsc
  ```

  Depending on the project and the vps, installing node dependencies can be a bit resource hungry.
  For example, if you are using a vps with 1GB of ram, you can install the dependencies with `npm install` and it will take a while.

   <!-- FIX: add the correct md or heading  -->

  Although there is a ci/cd technique in order to lessen the resource usage. check out [ci/cd technique using github action]

### Changing project file permissions

In order for the web server to access the project files for reading and writing(like when user upload an image or a file), you need to change the file permissions.

Instructions:

1. go to the project directory (`cd /var/www/html/project-name`)

2. then change the ownership of the project directory
   with the following command:

   ```bash
   sudo chown -R $USER:www-data .
   ```

   This command means :

   - chown : change the owner of the file or directory
   - `-R`: recursive
   - $USER : (value before the `:`) the user
   - www-data :(value after the `:`)the group of the user
   - . : the current directory

   this is needed so that the user and the group www-data(nginx/apache) now have permission for the current project.

3. then change the permission for the files
   with the following command:

   ```bash
   sudo find . -type f -exec chmod 664 {} \;
   ```

   this command means:

   - find : search for files
   - . : the current directory
   - -type f : only search for files
   - -exec : execute the following command for each file found
   - chmod 664 : change the permission of the file to 644
     - 6: read and write for owner(user)
     - 6: read and write for group
     - 4: read and write for others
   - {} : the file found
   - \; : end of the command

   > [!NOTE]
   > if you're using sqlite, you probably need to change the permission of the file to 660 so that the web server can only write

4. also change the permission for the directories
   with the following command:

   ```bash
   sudo find . -type d -exec chmod 755 {} \;
   ```

   - -type d : only search for directories

5. give permission to the web server to access cache and storage

   ```bash
   sudo chgrp -R www-data storage bootstrap/cache

   sudo chmod -R ug+rwx storage bootstrap/cache
   ```

   - chgrp : change the group of the file or directory
   - -R: recursive
   - www-data : the group of the user
   - storage and bootstrap/cache : the directory

   - chmod : change the permission of the file or directory
   - -R: recursive
   - ug+rwx : give permission to the user and group (ug) to read and write and execute

### connecting project to server block

When using nginx the sites are located in the `/etc/nginx/sites-available` directory.

If you install nginx and tried to open your vps ip in the browser the `default` file in that directory is the welcome page site

So to add our site we have to create and open the file with the name of our website.

code instruction:

1. `cd /etc/nginx/sites-available`

2. ````bash
      sudo vim <website-name>
      ```
   ````

   - <website-name> : the name of the website that matches your domain name or subdomain.

3. setup the server block

   Go to laravel documentation and search for nginx server websites configuration.

   alternatively, you can use this:

   ```bash

   server {
   listen 80;
   listen [::]:80;
   server_name odio.tingexe.cc;
   root /home/forge/domain.com/public;

   add_header X-Frame-Options "SAMEORIGIN";
   add_header X-Content-Type-Options "nosniff";

   index index.php;

   charset utf-8;

   location / {
   try_files $uri $uri/ /index.php?$query_string;
   }

   location = /favicon.ico { access_log off; log_not_found off; }
   location = /robots.txt  { access_log off; log_not_found off; }

   error_page 404 /index.php;

   location ~ \.php$ {
   fastcgi_pass unix:/var/run/php/php8.3-fpm.sock;
   fastcgi_param SCRIPT_FILENAME $realpath_root$fastcgi_script_name;
   include fastcgi_params;
   fastcgi_hide_header X-Powered-By;
   }

   location ~ /\.(?!well-known).* {
   deny all;
   }
   }
   ```

   - Then change and the `server_name` to your domain name or subdomain.

   - and root to the project directory.(/var/www/html/project-name)

   > [!TIP]
   > If you're using your domain for the server_name also add the www version of the domain for better coverage (www.example.com)

4. create a symlink to sites-enabled directory

   in order to activate the site you have to create a symlink to the sites-enabled directory

   ```bash
   sudo ln -s /etc/nginx/sites-available/<website-name> /etc/nginx/sites-enabled
   ```

   > [!WARNING]
   > make sure to use the full path of the directory

   then confirm it by going to the sites-enabled directory and check if the file is there

   ```bash
   cd /etc/nginx/sites-enabled
   ls -la
   ```

   you should see the file with the name of your website and arrow pointing to the symlink which is in sites-available directory

5. test the nginx configuration

   go to your project directory and run the following command:

   ```bash
   sudo nginx -t
   ```

   You should see the following message:

   ```bash
   nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
   nginx: configuration file /etc/nginx/nginx.conf test is successful
   ```

6. restart nginx

   ```bash
   sudo systemctl restart nginx
   ```

7. [connect your project to your domain](#pointing-dns)

   After adding the domain to your vps and you try to connect to it,
   if it returns a bad gateway error(502).

   Check your php extension `php8.3 -m`
   if you found nothing, you need to install the php fpm extension

   ```bash
   sudo apt install php8.3-fpm
   ```

   then restart the system ctl with :

   ```bash
   sudo systemctl restart php8.3-fpm
   ```

### Setting up ssl certificate

if your app is pointing to a cloudflare domain, most likely you're using the proxy to have a ssl/tls with client and domain. However, to ensure that you have an end-to-end encryption, it is a good idea to setup an ssl from vps to domain(cloudflare)

the popular option is to use certbot:

```bash
sudo apt install certbot python3-certbot-nginx
```

- Issue certificate for your website

  go to your project directory and run:

  ```bash
  sudo certbot --nginx -d <your-domain>.com
  ```

  there will be a prompt to enter your email and questions like:
  Terms of service (choose `Y`) and offer to share you email with the Electronic Frontier Foundation (better choose : `N`)

With that You can now have a working domain that is secured and in vps

## Other known issues

if you try to `git pull origin main` and get an error like a fatal error. This occurs when the ssh agent on your server stopped or expired at some point.

To resolve this, you can create a config file in your .ssh directory to avoid interruption for ssh agent.

```bash
cd ~/.ssh/

# create a config file
sudo vim config

# add the following content
User git
Hostname github.com
IdentityFile ~/.ssh/id_github
TCPKeepAlive yes
IdentitiesOnly yes
ServerAliveInterval 60
```

then change the ownership of the file

```bash
sudo chown <user> config
```

For the updated and detaild commands for vps setup check out [this github](https://github.com/glennraya/server-setup/blob/main/Commands.txt)
