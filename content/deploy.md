---
title: "Deploying laravel over VPS"
date: 2025-01-09
description: "a notes regarding deployment of application through ssh"
tags:
  - laravel
  - php
  - vps
---

# Using hetzner to deploy application

## Choosing OS

Normally a linux os and the most common is Ubuntu

- after choosing create the root password and add ssh key(try to find out how)

  Instructions:

  1.  run this command (need openssh to be installed)

  ```bash
  ssh-keygen it ed25519 -C "your_email@example.com"
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

  ### After install

1.  Add a user account after deploying the vps as working as a root user have a security risk

    use this cmd to add a user

    ```bash
    adduser <username>
    ```

    and fill up the necessary info like passwords

2.  Add the user in the sudo group so that you wont need a sudo permission

    use this cmd :

    ```bash
    usermod -aG sudo <username>
    ```

3.  switch to the newly `sudo` access user with

```bash
su - <usernamed>
```

then test it with

```bash
sudo ls /
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

## hardening ssh

In order to avoid a bruteforce ssh attack, password must be disabled

1. (in local machine)

   ```bash
   ssh-copy-id <user>@<vps-id-addr>
   ```

   youll have to authenticate through this command and with this will enable a keybased authentication

   then test it by login through ssh again and this time it should not ask for password again

2. editing the sshd_config

```bash
sudo vim /etc/ssh/ssh_config
```

here you can edit the following:

- uncomment `PasswordAuthentication yes` and change it to `PasswordAuthentication no`
- find `PermitRootLogin yes` to `PermitRootLogin no` (remove the ability to login as root)
- find `UsePAM yes` to `UsePAM no`

> [!NOTE]
> also check /etc/ssh/ssh_config.d/
> for another file like `50-cloud-init.conf`

if it exists also apply the necessary steps above to it (or remove it )

3. reload systemctl

to apply the changes reload the ssh service with:

```bash
sudo systemctl reload ssh
```

3. test
   to test you can logout the ssh and try logging in with root user `root@<vps-id-addr>`
   if you weren't able to login meaning it is successful

4. (optional) change the port to something else
   in the `/etc/ssh/ssh_config`

you can change the port to something else to avoid the attacks from automated scanners

you can do this by finding and uncommenting the `port 22` to like `port 8000`

> [!NOTE]
> make sure to take note of the changed port
