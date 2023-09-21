+++
author = "Lonus"
title = "How to install & host Ghost with Cloudflare Argo tunnel"
date = "2022-06-15"
description = "A guide to teach you how to run Ghost and Cloudflare Argo Tunnel"
tags = [
    "guide",
    "self-hosted",
    "linux",
    "cms",
]
categories = [
    "Guide",
    "Self-hosting",
    "Ghost",
    "Cloudflare",
    "Linux",
]
series = ["Ghost"]
aliases = ["migrate-from-ghost"]
image = "cloudflared-ghost.png"
+++

<!--adsense-->

# Overview

You are here for a reason

- You want to self host your own site but you don't have a public IP and you can't pay your ISP for one because you're poor.

# Requirements:

- Nodejs (v16x)

- NGIX

- Ubuntu 20.04

- MySQL

- Cloudflare account.

- Domain name.

- Cloudflared tunnel service.

- Ghost

- Certbot

> **notice:** If you doing in on a old machine keep in mind that you might have a bunch of error and crashes one of them its explained at the end.

## FIRST THING

```bash
sudo apt update
sudo apt upgrade
```

## INSTALLING NGINX

```bash
sudo apt install nginx
```

Once is installed sometimes it doesn't automatically run `NGINX` so lets check if its running if not lets enable it

```bash
# to check if its running run
sudo systemctl status nginx

# if the output doesn't say its running run
sudo systemctl start nginx
```

## INSTALLING NODEJS

`sudo apt-get install nodejs` doesn't install out of the box the supported version for Ghost so don't do it before running the following command.

```bash
curl -sL https://deb.nodesource.com/setup_16.x | sudo -E bash
sudo apt install -y nodejs
```

You can also install it with `nvm` .

Install `nvm` if you don't have it :

```bash
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.1/install.sh | bash
```

after you have it installed you can now install nodejs by running:

```bash
nvm install --lts # installs the supported version
# It automatically changes to the version you installed
# if you want to check the installed version run
nvm ls
# and itll show you current installed versions and the one
# is in use.
```

If by the time you are doing this the `LTS` of nodejs changed or the one supported in Ghost you can do:

```bash
nvm ls-remote # it'll show you all the node versions available
# Select the one that ghost supports
nvm install v16.5.1 # change the version to the one you want
# and
nvm use v16.5.1 # again type the one you just installed
```

## INSTALLING MYSQL

To install it run the following:

```bash
sudo apt install mysql-server
```

### Set up MySQL user that `Ghost` is going to use

Enter MySQL:

```bash
sudo mysql -u root -p
# It'll prompt you to enter your root password
```

Now update the user with this command. Make sure to replace `password` with your own secure password **KEEP THE QUOTE MARKS**

```sql
ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY 'password';
```

Doing so it should prompt you with the following:

```sql
Query OK, 0 rows affected (0.18 sec)
```

The password and user are all set up now you can exit MySQL

```sql
quit or \q
```

# INSTALLING GHOST-CLI AND SETTING GHOST FOLDER

To install Ghost-CLI do :

```bash
sudo npm install ghost-cli@latest --location=global
```

For now lets set up where our Ghost site its going to be installed

Create the folder inside the `/var/www` path. you can install it wherever you want but this is the preferred path so `NGINX` can detect your Ghost site.

```bash
sudo mkdir -p /var/www/your-site-name
```

Give the folder the proper permissions to your user :

```bash
sudo chown $USER:$USER /var/www/your-folder-site-name
```

```bash
sudo chmod 775 /var/www/your-folder-site-name
```

## INSTALL GHOST

Move to the folder where your Ghost site it's going to be installed

```bash
cd /var/www/your-folder-site-name
```

Once you are inside the folder type :

```bash
ghost install
```

Let it run and when itll prompt you with:

```bash
Enter your blog URL: # put your domain name don't use localhost unless your installing in development mode
Enter your MySQL hostname: #keep it in localhost so just press enter
Enter your MySQL username: root # if you used a different user change it to it.
Enter your MySQL password: # use the same password you put when we set upt mysql
Enter your Ghost database name: # just press enter
Setting up "ghost" system user
? Do you wish to set up "ghost" mysql user? (Y/n) : Y
Do you wish to set up Nginx? (Y/n) : Y # so it automatically creates the nginx config files
Setting up SSL : # press no since we are gonna add manually the certificates to the config file later
Setting up Systemd: # press yes if you want your site to run on boot
Do you want to start Ghost?: N #press no since we need to set up more things before we run the site
```

# SETTING UP CERTBOT AND LETSENCRYPT CERTIFICATES

If your Ubuntu installation comes with a certbot package preinstalled you might need to remove before doing this

```bash
sudo apt-get remove certbot
```

Execute the following to make sure you have the latest version of snapd installed

```bash
sudo snap install core; sudo snap refresh core
```

## Installing CERTBOT

```bash
sudo snap install --classic certbot
```

## Make the command available

Execute the following instruction on the command line on the machine to ensure that the `certbot` command can be run.

```bash
sudo ln -s /snap/bin/certbot /usr/bin/certbot
```

## Installing NGINX and CLOUDFLARE plugins

### Installing NGINX plugin

In order to automatically put the certificates in the site config file we must need nginx certbot plugin, so to install it run :

```bash
sudo apt install certbot python3-certbot-nginx
```

### Installing and setting up cloudflare plugin

And since where we installing the site the IP isn't a public one we need to get the certificates by doing dns challenges to cloudflare so lets install the certbot plugin first :

```bash
sudo apt install certbot python3-certbot-dns-cloudflare
```

---

### Generate an API token for CloudFlare

> At this point im assuming you have already added your domain name to Cloudflare as your DNS provide.

---

- Navigate to [Cloudflare | Web Performance &amp; Security](https://dash.cloudflare.com/profile/api-tokens)
- Next to "Global API Key", click "View".
- Copy the token that's shown. We will need this token in the next step.

### Store the CloudFlare API token on the server

---

Create a file to hold your secret API key like this:

```bash
# create the folder where the file its going to be stored
sudo mkdir -p /etc/letsencrypt/secrets
# create the file
sudo touch /etc/letsencrytp/secrets/cloudflare.ini
# give the file the permissions
sudo chown root:root /etc/letsencrypt/cloudflare.ini
sudo chmod 600 /etc/letsencrypt/cloudflare.ini
```

The insert your API details:

```bash
sudo nano /etc/letsencrypt/secrets/cloudflare.ini
```

Put your Cloudfalre account email and API key that you got from the previous step:

```bash
# Cloudflare API credentials used by Certbot
dns_cloudflare_email = # the email you use to login to cloudflare
dns_cloudflare_api_key = # the API you got before
```

### Request your certificate with certbot

---

```bash
sudo certbot certonly --dns-cloudflare --dns-cloudflare-credentials /etc/letsencrypt/secrets/cloudflare.ini
```

Enter the email you are going to be using for the renewal notifications and accept the ToS then add the domain you want to get the certificates for (the one you are going to use for the ghost site).

```bash
Enter email address (used for urgent renewal and security notices) (Enter 'c' to
cancel): # your email here
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
Please read the Terms of Service at
https://letsencrypt.org/documents/LE-SA-v1.2-November-15-2017.pdf. You must
agree in order to register with the ACME server at
https://acme-v02.api.letsencrypt.org/directory
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
(A)gree/(C)ancel: a

- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
Would you be willing to share your email address with the Electronic Frontier
Foundation, a founding partner of the Let's Encrypt project and the non-profit
organization that develops Certbot? We'd like to send you email about our work
encrypting the web, EFF news, campaigns, and ways to support digital freedom.
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
(Y)es/(N)o: n
Please enter in your domain name(s) (comma and/or space separated)  (Enter 'c'
to cancel): your-site-domain
Obtaining a new certificate
Performing the following challenges:
dns-01 challenge for your-site-domain
Waiting 10 seconds for DNS changes to propagate
Waiting for verification...
Cleaning up challenges

IMPORTANT NOTES:
 - Congratulations! Your certificate and chain have been saved at:
   /etc/letsencrypt/live/your-site-domain/fullchain.pem
   Your key file has been saved at:
   /etc/letsencrypt/live/your-site-domain/privkey.pem
   Your cert will expire on 2022-09-11. To obtain a new or tweaked
   version of this certificate in the future, simply run certbot
   again. To non-interactively renew *all* of your certificates, run
   "certbot renew"
 - Your account credentials have been saved in your Certbot
   configuration directory at /etc/letsencrypt. You should make a
   secure backup of this folder now. This configuration directory will
   also contain certificates and private keys obtained by Certbot so
   making regular backups of this folder is ideal.
 - If you like Certbot, please consider supporting our work by:

   Donating to ISRG / Let's Encrypt:   https://letsencrypt.org/donate
   Donating to EFF:                    https://eff.org/donate-le
```

### Telling CERTBOT to add the certificates to the nginx config file of your site.

---

To make certbot add them automatically run :

```bash
sudo certbot --nginx
```

If after running the commad you get this error :

```bash
Plugins selected: Authenticator nginx, Installer nginx
No names were found in your configuration files. Please enter in your domain
name(s) (comma and/or space separated)  (Enter 'c' to cancel):
```

Theres a reason why, as you can read Certbot is not capable of detecting the `conf` of your site that Ghost automatically added to `sites-available` and `Certbot` is looking for a `conf` file with a `domain name` .

So a way to fix this is modify the `nginx,conf` file and include the path to `sites-enabled`

```bash
# modify the nginx.conf file
sudo nano /etc/nginx/nginx.conf
```

And add the following line inside the `http {}`block

```bash
include /etc/nginx/sites-enabled/*;
```

Save it and reload NGINX.

```bash
sudo systemctl reload nginx
```

So now if we run the `Certbot`command we should get the following:

```bash
# command to put the certificates in the conf file
sudo certbot --nginx
# Output prompt
Plugins selected: Authenticator nginx, Installer nginx

Which names would you like to activate HTTPS for?
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
1: yourdomainname
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
Select the appropriate numbers separated by commas and/or spaces, or leave input
blank to select all options shown (Enter 'c' to cancel): 1
Cert not yet due for renewal

You have an existing certificate that has exactly the same domains or certificate name you requested and isn't close to expiry.
(ref: /etc/letsencrypt/renewal/yoursite.conf)

What would you like to do?
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
1: Attempt to reinstall this existing certificate
2: Renew & replace the cert (limit ~5 per 7 days)
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
Select the appropriate number [1-2] then [enter] (press 'c' to cancel): 1
Keeping the existing certificate
Deploying Certificate to VirtualHost /etc/nginx/conf.d/yoursite.conf

Please choose whether or not to redirect HTTP traffic to HTTPS, removing HTTP access.
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
1: No redirect - Make no further changes to the webserver configuration.
2: Redirect - Make all requests redirect to secure HTTPS access. Choose this for
new sites, or if you're confident your site works on HTTPS. You can undo this
change by editing your web server's configuration.
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
Select the appropriate number [1-2] then [enter] (press 'c' to cancel): 2
Redirecting all traffic on port 80 to ssl in /etc/nginx/conf.d/yoursite.conf

- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
Congratulations! You have successfully enabled https://yourdomainname

You should test your configuration at:
https://www.ssllabs.com/ssltest/analyze.html?d=yourdomain
```

If you check your site`conf` file the certificates should be applied and all the extra configuration too.

> For more information on how to renew the type `certbot --help` or type `sudo certbot renew` this wil renew all your certificates so if you have multiple certificates for different domains and you want to renew a specific one run `certbot certonly --force-renew -d yourdomain.com`.

## Setting up Cloudflare Tunnel

---

### Install the Cloudflared service:

**NOTE**

> A little note on why i don't use the `Zero Trust Dashboard` and I do this manually.
>
> 1 - `Zero Trust dasboard` lacks of configuration options compared to when you create the tunnel manually.
>
> **Example:**
>
> If you install the service and create the tunnel through the `Zero Trust Dashboard` pointing to `localhost:2368` where Ghost server is running and when you the service you are going to encounter into a problem of `ERRO_TO_MANY_REDIRECTS` after many hours trying to know why was this happening I found out that setting `noTLSVerfiy` to `true` the site loads And the only way to do this is when you create the tunnel manually since the `Zero Trust Dashboard` doesn't have this option.

First you have to switch to the `root` user

```bash
# to change to the root user simply type
sudo su
```

Once you are in the `root` user run the following :

```bash
wget -q https://github.com/cloudflare/cloudflared/releases/latest/download/cloudflared-linux-amd64.deb && dpkg -i cloudflared-linux-amd64.deb
```

This will download the cloudflared package install it.

### Authenticate cloudflared

---

```bash
cloudflared tunnel login
```

Running this command will:

- Open a browser window and prompt you to log in to your Cloudflare account. After logging in to your account, select your domain.(If after login in it doesn't redirect you the correct page just click on the link that its in your terminal.)
- Generate an account certificate, the [cert.pem file](https://developers.cloudflare.com/cloudflare-one/connections/connect-apps/install-and-setup/tunnel-useful-terms/#cert-pem), in the [default `cloudflared` directory](https://developers.cloudflare.com/cloudflare-one/connections/connect-apps/install-and-setup/tunnel-useful-terms/#default-cloudflared-directory).

### Creating the tunnel

---

To create the tunnel run :

```bash
# Create the tunnel
cloudflared tunnel create <NAME>
# Running this command will:
# 1 - Create a tunnel by establishing a persistent relationship between
# the name you provide and a UUID for your tunnel. At this point,
# no connection is active within the tunnel yet.
# 2 - Generate a tunnel credentials file in the default cloudflared directory.
# 3 - Create a subdomain of .cfargotunnel.com.
#From the output of the command, take note of the tunnel’s UUID and
# the path to your tunnel’s credentials file.

# Check that the tunnel was creating
cloudflared tunnel list
# It'll prompt you with the tunnels that you have created including
# the ones that you created through zero trust dashboard.
```

After creating the tunnel it's time to create the `config.yml` file inside the `.cloudflared` folder this is where you tell the tunnel where and what service to look for.

```bash
# move the .cloudflared folder
cd ~/.cloudflared
# If you LS that directory you will see two files
ls
cert.pem the-tunnel-you-created-UUID.json

# create the config file
touch config.yml


# Open it and add the following
nano config.yml
```

Add the following to the config file

```vim
tunnel: your-tunnel-UUID
credentials-file: /home/$USER/.cloudflared/your-tunnel-UUID.json

ingress:
  - hostname: your-domain-name
    service: https://localhost:443
    originRequest:
      noTLSVerify: true
  - service: http_status:404
```

- **hostname:** Here add the domain you are going to use. Make sure is the same you put in your Ghost setup.

- **service:** Since we added SSL certificates and NGINX it's in charge of redirecting everything we don't need to put the `localhost:2368` where Ghost server is normally running.

- **originRequest:**

  - **noTLSVerfiy: true** # Don't remove this or you are going to get the `ERROR_TO_MANY_REDIRECTS` error.

### Route the traffic

---

Now assign a CNAME record that points traffic to your tunnel.

```bash
cloudflared tunnel route dns <UUID or NAME> <your-domain>
```

### Run the tunnel

---

Run the tunnel to see if it works

```bash
cloudflared tunnel run <UUID or NAME>
# you can stop it by doing ctrl+c
```

If you want to run the tunnel as a service follow Cloudflare's guide [Run as a service on Linux · Cloudflare Zero Trust docs.](https://developers.cloudflare.com/cloudflare-one/connections/connect-apps/run-tunnel/as-a-service/linux/)

## Running GHOST

---

If you added Ghost to systemd, all you have to do is start your service

```bash
# The name of the service might be different so look for it in
# /etc/systemd/system/ or /lib/systemd/systemd/
sudo systemctl start ghost_yourdomain  # you can also stop/disable/enable
# or
sudo service ghost_yourdomain start # you can also stop/disable/enable
```

But if you want to run it and see if everything is working properly move to where you have installed it

```bash
cd /var/www/yoursite
```

And run ghost with :

```bash
ghost run
```

This will launch ghost in debug mode and you will be able to see where it fails. To end the process simply do `ctrl + c`.

### Known issues

---

If you are running Ghost on an older PC its probably that at the moment you want to upload any image (profile picture or post image) Ghost will silently crash and this is probably because your CPU doesn't support `SSE4.2`.

This due to the `sharp` dropped support on older CPU that are running below `SSE4.2`

So a Fix is disabling the `imageOptimization` to do this add the following to your `conf.production.json`

```json
  "imageOptimization": {
    "resize": false,
    "srcsets": false
  }
```

After doing you should be able to upload images to your site **BUT** since you disabled `imageOptimization` ghost is going to crash after you upload the `Publication cover` image so keep in mind that.
