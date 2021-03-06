# Getting started

*Please do not attempt to setup this project if you have no IT knowledge whatsoever! And do not try to set this up on Windows! You're better off with a Virtual Machine containing Ubuntu. You can also try the [Raspberry Pi image](getting-started-rpi-image) (which is way easier to setup)*

**Articles which are relevant for this project:**

* [General introduction to WhatsSpy Public](https://maikel.pro/blog/en-whatsapp-privacy-options-are-illusions/)
* [In-depth about the WhatsApp privacy problem](https://maikel.pro/blog/en-whatsapp-privacy-problem-explained-in-detail/) **(WhatsApp acts like it's part of their service)**
* [Current status of the problem (15 april 2015 update)](https://maikel.pro/blog/en-status-of-whatsapp-privacy-problem/)


**Important links:**

[Project page](https://gitlab.maikel.pro/maikeldus/WhatsSpy-Public/wikis/home) | [Screenshots](https://gitlab.maikel.pro/maikeldus/WhatsSpy-Public/wikis/home#overview) | [Other installation options](https://gitlab.maikel.pro/maikeldus/WhatsSpy-Public/wikis/home#installling-whatsspy-public-on-your-linux-machine-vps-linux-raspberry-pi)

## 1) Check requirements

* Secondary WhatsApp account (phonenumber that doesn't actively uses WhatsApp, because you can't receive messages when the tracker is online).
* Linux Server/VPS/Raspberry Pi/Linux Desktop that runs 24/7 (tested on Debian/Ubuntu) (**Do not use Windows**, use a [virtual machine instead](http://www.oracle.com/technetwork/server-storage/virtualbox/downloads/index.html)).
* Full command line access to your machine (a simple PHP webhoster won't work).

## 2) Manual install (Debian/Ubuntu)

### 2.0) Support

In case you find it hard to setup this project you can follow this **[video tutorial](https://gitlab.maikel.pro/maikeldus/WhatsSpy-Public/wikis/how-to-setup-video)** and use the **[troubleshooting page](troubleshooting)** when you get stuck.

### 2.1) Required packages and config

Install the following packages:
```
sudo apt-get install postgresql nginx php5 php5-cli php5-curl php5-fpm php5-pgsql git-core screen
```

**1)** Verify that your PHP-FPM installation uses the Unix socket:
```
sudo nano /etc/php5/fpm/pool.d/www.conf
```

The following line should be in the config file (and **NOT** `listen = 127.0.0.1`):
```
listen = /var/run/php5-fpm.sock
```

If you have changed it make sure you restart it: `sudo service php5-fpm restart`.

**2)** Make sure that your PostgreSQL allows local connections for the user `postgres`:

```

sudo nano /etc/postgresql/9.1/main/pg_hba.conf
``` 
*This file might be in a different location (like `/etc/postgresql/9.4/main/pg_hba.conf`). You can execute `cd /etc/postgresql/ && ls` to see which version you have. Make sure you have at least Postgres 9.0*

Search for the following line:
```
local   all             postgres                                peer
```

and change it to:
```
local   all             postgres                                trust
```
Reload the configuration by using `sudo service postgresql reload`.

### 2.2) Download WhatsSpy Public

Make a folder on your machine: `/var/www/whatsspy/` (with `mkdir`). After this download WhatsSpy Public:
```
cd /var/www/whatsspy/
git init
git remote add origin https://gitlab.maikel.pro/maikeldus/WhatsSpy-Public.git
git pull origin master
```
*(Please note that SSH and HTTP does not work on this Gitlab, only HTTPS)*

### 2.3) Retrieve the `secret` for a secondary WhatsApp account
WhatsSpy Public **requires** a secondary WhatsApp account. Once the tracker is started, you will not be able to receive any messages over WhatsApp for this phone number. You need to register at WhatsApp to retrieve a `secret` which you will need later when setting up WhatsSpy Public.

   * Execute `cd /var/www/whatsspy/api/whatsapp/ && sudo php registerTool.php` (use `chown -R <your-username> /var/www/whatsspy` in case you get an "Could not open ..." error).
   * Enter your phonenumber that you want to use for the WhatsSpy Public tracker.
      * `number` needs to be <countrycode><phonenumber> without any prefix 0's. *0031 06 120..* becomes *31 6 120..* (no 0's prefix for both the countrycode and phonenumber itself).
      * `number` may only contain digits. Spaces, plus or any other special character are NOT accepted. *Example: 316732174*
   * Request activation via `SMS` and wait for a SMS to arrive at the phone  (try `voice` if you did not get the SMS).
   * Enter the retrieved code in the script without any dashes (only the digits!).
   * Write down the `secret` (it's the one-line of strange characters ending with an =).

*There are [two other methods described here](ways-of-getting-the-secret) in case this method does not work.*

*Did you get `[status] => fail [reason] => bad_param` after choosing SMS or voice? Make sure you entered the number correctly, after that check [this solution](https://gitlab.maikel.pro/maikeldus/WhatsSpy-Public/wikis/ways-of-getting-the-secret#1-registration-via-the-registertool-php-script).*

**Make sure that you do not use this WhatsApp account once the tracker is running.**

### 2.4) Setup database

Login into the database:
```
psql -U postgres
```
*(In case you get an authentication error you did not setup the PostgreSQL authentication correct)*

Create the WhatsSpy user in the database (choose a password):
```
CREATE ROLE whatsspy LOGIN
  PASSWORD 'CHOOSEAPASSWORD'
  NOSUPERUSER INHERIT NOCREATEDB CREATEROLE REPLICATION;
```
*(Using another database username means you need to change every reference in `whatsspy-db.sql`)*

Create database:
```
CREATE DATABASE whatsspy
  WITH OWNER = whatsspy
       ENCODING = 'UTF8'
       TABLESPACE = pg_default
       CONNECTION LIMIT = -1;
```

Grant the rights:
```
\connect whatsspy
GRANT ALL ON DATABASE whatsspy TO whatsspy;
GRANT ALL PRIVILEGES ON ALL TABLES IN SCHEMA public TO whatsspy;
```

Quit the database:
```
\q
```
*(or use `Ctrl`+`Z` in case you can't type `\q`)*

Now it is time to insert the WhatsSpy Public database in PostgreSQL. Execute the following commands:
```
cd /var/www/whatsspy/api/
psql -U postgres -d whatsspy -f whatsspy-db.sql
``` 

### 2.4) Setup the config

Copy `config.example.php` to `config.php` located at `/var/www/whatsspy/api/` and fill in the following details: 

* PostgreSQL password correctly in `$dbAuth`.
* Insert your `number` and `secret` in `$whatsappAuth`. 
  * `number` needs to be <countrycode><phonenumber> without any prefix 0's. 0031 06 xxx becomes 31 6 xxx (no 0's prefix for both the country code and phonenumber itself).
  * `number` may only contain digits. Spaces, plus or any other special character are NOT accepted. *Example: 316732174 is correct*
  * `secret` You obtained this in the chapter *2.3) Retrieve the `secret` for a secondary WhatsApp account*.
* Set the correct timezone of the place where you are in `date_default_timezone_set('MYTIMEZONE');` [(list of timezones)](http://php.net/manual/en/timezones.php).
* Choose a password for the login later on by changing `$whatsspyPublicAuth = 'whatsspypublic';` to something else.
* In case you did **not** install WhatsSpy Public in `/var/www/whatsspy/`, set the path for `$whatsspyProfilePath`.
  * `$whatsspyProfilePath` is the absolute path for the system to store the profile pictures. For example `/var/www/whatsspy/images/profilepicture/` (default setting), `/var/www/other-dir/images/profilepicture/`. Don't forget the last `/`!

### 2.5) Correct file rights

The tracker needs read/write acces in the folder `$whatsspyProfilePath`, `api/whatsapp/src/wadata/`.

```
sudo chown www-data:www-data -R /var/www/whatsspy/
sudo chmod 775 -R /var/www/whatsspy/
sudo chmod 760 -R /var/www/whatsspy/api/whatsapp/src/wadata/
# Did you change $whatsspyProfilePath? chmod 760 this path instead of the one stated below here:
sudo chmod 760 -R /var/www/whatsspy/images/profilepicture/
```
*(In case you get any `Permission Denied` errors you can execute the same commandos with 770 for debugging)*

### 2.6) Configure web server

You need to restrict access to WhatsSpy Public and the API of WhatsSpy Public from unauthorized web access. We need to update your Nginx configuration to restrict access:

`sudo nano /etc/nginx/sites-available/default`

And make sure your configuration looks like:
```
server {
    listen 80;
    root /var/www;
    index index.html index.php;

    
    location /whatsspy/api/whatsapp/ {
        deny all;
        return 404;
    }

    location /whatsspy/images/profilepicture/ {
        deny all;
        return 404;
    }

    location ~ ^/whatsspy/api/((?!index\.php).+) {
        deny all;
        return 404;
    }

    location ~ /whatsspy/api/$ {
        index index.php;
    }

    location ~ \.php$ {
        try_files $uri =404;
        fastcgi_pass unix:/var/run/php5-fpm.sock;
        fastcgi_index index.php;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        include fastcgi_params;
    }

}
``` 
In case you installed your WhatsSpy Public in a other place you need to edit the paths slightly. Make sure you reload the configuration by executing `sudo service nginx reload`.

**Note:** Some operating systems might have Apache already running. Make sure you first stop Apache and after that start Nginx (`sudo service apache stop && sudo service nginx restart`).

**Or if you use Apache (otherwise ignore):** Check [this page with the apache config](https://gitlab.maikel.pro/maikeldus/WhatsSpy-Public/wikis/getting-started-apache-config)

## 3) Importing users

If everything went well you can now access the WhatsSpy Public interface through your webserver (via `http://<your-ip>/whatsspy/`) (default password is `whatsspypublic`). At this point you need to import users that you want to track ([Troubleshooting](troubleshooting)):

* Either add any contact manually by using "Add contact by phonenumber".
* Or use "import google Contacts" which is an script that retrieves all your Google Contacts and gives an SQL statement which insert all users into the database.

**Notice:**
* Once you have inserted these users they won't show up automatically. They need to be verified by the tracker which is not running yet.

## 4) Starting the tracker
Once you have populated your database with some users, you can start the tracker.

1. Execute `screen`
3. cd to the install of the Whatsspy (`cd /var/www/whatsspy/api/`) and execute `` `which php` tracker.php`` (it is important that you first `cd` and then execute the cmd, otherwise paths will be incorrect).
   * Make sure you are in the WhatsSpy Public API dir (execute `cd /var/www/whatsspy/api/`).
   * In case you get a "Permission denied" error try running it as: ``sudo -u www-data `which php` tracker.php``
4. If all runs well it starts spamming information about privacy options and polls.
5. It keeps polling every second and outputs any statuses on the screen.
6. You can exit the screen by using `Ctrl+a` and after that `Ctrl+d` (detaching the screen) in your terminal/Putty.

**In case of `[error] Tracker Exception! Login failure!` check [troubleshooting](https://gitlab.maikel.pro/maikeldus/WhatsSpy-Public/wikis/troubleshooting#troubleshooting)**

## 10) Optional steps / handy things to know

* You can enable notifications via *NotifyMyAndroid*, *LiveNotifier* or even WhatsApp. For this you need to edit your `config.php` (like before) and change entries in `$whatsspyNotificatons`. After saving your change restart the tracker.
* If you want to share profiles via the internet you need to make sure your installation is accessible from the internet.
* If you want to log the tracker session to a log file you can call ``sudo -u www-data  `which php` tracker.php | tee /var/log/tracker.log`` instead of step **3** in **Starting the tracker** (and you must install `tee`).

Please check the [FAQ](FAQ).