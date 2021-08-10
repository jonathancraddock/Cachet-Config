# Notes on Cachet Config

See: [Cachet](https://github.com/cachethq/Cachet)

Contents:  
* [Installing On Ubuntu 20](#installing-on-ubuntu-20)  
* [Page Design](#page-design)  
* [API Examples](#api-examples)  
* [API NodeRED](#api-nodered)
* [References](#references)  

## Installing on Ubuntu 20

I don't see many people complaining about the installation instructions, so maybe it's just me, but I tried multiple times with the official instructions, and with guides on various blog posts, and then tried the Docker version. Basically nothing worked as expected. However, the guide on https://websiteforstudents.com/install-cachet-status-platform-on-ubuntu-16-04-18-04-with-apache2-mariadb-and-php-7-2/ does appear to work ok, and I've summarised my steps below.

Started with a clean Ubuntu 20 LTS install. I created a single sudo user and ran the usual updates. Took a snapshot here, in case I needed to revert the setup, again!

> There's a useful guide here: https://www.digitalocean.com/community/tutorials/how-to-install-the-apache-web-server-on-ubuntu-20-04 and do remember to *enable* the services, as you presumably want them to auto start after a reboot.

#### Apache

```bash
sudo apt update
sudo apt install apache2
curl localhost
```
^- *the default Apache page should be served*

#### Git and cURL

They are probably already installed, but just in case:

```bash
sudo apt install curl git
```


#### MariaDB

```bash
sudo apt-get install mariadb-server mariadb-client
sudo mysql_secure_installation
```

Secure the database as you wish.

```bash
sudo mysql -u root -p
exit;
```
^- *confirm you can login*

#### PHP

There is a variety of conflicting advice around PHP versions. I tried 7.2, as per this guide, and it appears to work ok.

```bash
sudo apt-get install software-properties-common
sudo add-apt-repository ppa:ondrej/php
sudo apt update
sudo apt install php7.2 libapache2-mod-php7.2 php7.2-common php7.2-mysql php7.2-gmp php7.2-curl php7.2-intl php7.2-mbstring php7.2-xmlrpc php7.2-gd php7.2-bcmath php7.2-imap php7.2-xml php7.2-cli php7.2-zip
```

Made the following suggested edits `sudo nano /etc/php/7.2/apache2/php.ini`.

```bash
file_uploads = On
allow_url_fopen = On
short_open_tag = On
memory_limit = 256M
upload_max_filesize = 100M
max_execution_time = 360
date.timezone = Europe/London
```

Restart Apache and (if you want to) confirm that PHP is ok.

```bash
sudo systemctl restart apache2
sudo nano /var/www/html/phpinfo.php
```

Add `<?php phpinfo( ); ?>` to the top of the file and `curl localhost/phpinfo.php`. It should be served as html.

#### Create the Database

```bash
sudo mysql -u root -p
CREATE DATABASE cachetdb;
CREATE USER 'cachetdbuser'@'localhost' IDENTIFIED BY '****';
GRANT ALL ON cachetdb.* TO 'cachetdbuser'@'localhost' IDENTIFIED BY '****' WITH GRANT OPTION;
FLUSH PRIVILEGES;
EXIT;
```

#### Cachet App

```bash
curl -sS https://getcomposer.org/installer | sudo php -- --install-dir=/usr/local/bin --filename=composer
cd /var/www/html
sudo git clone -b 2.4 --single-branch https://github.com/cachethq/Cachet.git cachet
sudo cp /var/www/html/cachet/.env.example /var/www/html/cachet/.env
```
^- *I'd normally install higher than the default html folder, but after all the preceeding hassle I was following their guide exactly...*

Edit the env file, `sudo nano /var/www/html/cachet/.env`.

```bash
DB_HOST=localhost
DB_DATABASE=cachetdb
DB_USERNAME=cachetdbuser
DB_PASSWORD=****
```

I believe it's bad practice to run Composer with sudo, there may be a warning? I accepted it with a 'Y'.

```bash
cd /var/www/html/cachet
sudo composer install --no-dev -o
```

Answer "no" to the two prompts on the Cachet install.

```bash
sudo php artisan key:generate
sudo php artisan cachet:install
```
^- *if you want to check, the key should now be written into the `.env` file.*

Reset ownerships and permissions.

```bash
sudo chown -R www-data:www-data /var/www/html/cachet/
sudo chmod -R 755 /var/www/html/cachet/
```
^- *Surely this is far too generous?! These were recommended by the install guide, but in production I'd question this owenership, and I think only the `cache` and `storage` folders need to be writable. Possibly also `database`, if you're using sqlite.*

#### Apache Conf

Create a conf file.

```bash
sudo nano /etc/apache2/sites-available/cachet.conf
```

Their suggestion:

```bash
<VirtualHost *:80>
     ServerAdmin cachet@example.com
     DocumentRoot /var/www/html/cachet/public
     ServerName cachet.example.com
     ServerAlias cachet.example.com

     <Directory /var/www/html/cachet/public>
          Options FollowSymlinks
          AllowOverride All
          Require all granted
     </Directory>

     ErrorLog ${APACHE_LOG_DIR}/error.log
     CustomLog ${APACHE_LOG_DIR}/access.log combined

     <Directory /var/www/html/cachet/public>
            RewriteEngine on
            RewriteBase /
            RewriteCond %{REQUEST_FILENAME} !-f
            RewriteRule ^(.*) index.php [PT,L]
    </Directory>
</VirtualHost>
```

Enable the site.

```bash
sudo a2ensite cachet.conf
sudo a2enmod rewrite
sudo systemctl restart apache2
```

#### Let's Encrypt

And secure the site.

```bash
sudo snap install --classic certbot
sudo ln -s /snap/bin/certbot /usr/bin/certbot
sudo certbot --apache
```

#### In Browser Config

You should now be able to complete the setup in your browser.

```
Cache Driver - database
Queue Driver - none
Session Driver - database
```
^- *the following driver selection works, but may not be optimal?*

-----

## Page Design

The file `/var/www/cachet/resources/views/index.blade.php` contains the Laravel blade template for the main page layout. For example, if you need the page sections to appear in a different order.

```php
@extends('layout.master')

@section('content')
@include('partials.modules.messages')
@include('partials.modules.status')
@include('partials.about-app')
@include('partials.modules.components')
@include('partials.modules.metrics')
@include('partials.modules.stickied')
@include('partials.modules.scheduled')
@include('partials.modules.timeline')
@stop

@section('bottom-content')
@include('partials.footer')
@stop
```
^- *eg/ if you wanted the "about" panel to display above the "status" message, etc.*

Highlight the "Maintenance" section:

```css
.section-scheduled {
  border: 1px solid #3929a8;
  border-radius: 4px;
  box-shadow: 0 4px 8px 0 rgba(0, 0, 0, 0.2), 0 6px 20px 0 rgba(0, 0, 0, 0.19);
}
```

-----

## API Examples

Update status of a component:  
```bash
curl -X PUT -H "Content-Type: application/json" -H "X-Cachet-Token: API-KEY-HERE" -d '{"status":1}' "https://cachet.example.com/api/v1/components/1"
```
^- *in this example, set component 1 to status of 'operational'*

### Components:
* `/api/v1/components` list all components
* `/api/v1/components/1` query specific component
* `/api/v1/components/groups` list all component groups
* `/api/v1/components/groups/1` components in specified group 

#### Component fields?
```text
"id": 1,
"name": "Component Name",
"description": "Description goes here",
"link": "",
"status": 1,
"order": 0,
"group_id": 1,
"enabled": true,
"meta": null,
"created_at": "2021-07-29 18:51:27",
"updated_at": "2021-08-03 12:19:47",
"deleted_at": null,
"status_name": "Operational",
"tags": []
```

#### Group fields?
```text
"id": 1,
"name": "Group Name",
"order": 0,
"visible": 1,
"collapsed": 0,
"created_at": "2021-07-29 18:58:49",
"updated_at": "2021-07-30 15:57:42",
"enabled_components": [],
"lowest_human_status": "Operational"
```

### Incidents:
* `/api/v1/incidents` list all incidents
* `/api/v1/incidents/1` query specific incident
* `/api/v1/incidents/1/updates` list updates on specified incident
* `/api/v1/incidents/1/updates/1` query specific update on specified incident

### Metrics
* `/api/v1/metrics` list all metrics  
* `/api/v1/metrics/1` query a specific metric

### Maintenance:
* (Don't know if this can be queried via their API?)

### General:
* `/api/v1/version` replies with version number
* `/api/v1/ping` replies with "pong"
* `api/v1/subscribers` (requires authentication)

-----

## API NodeRED

### Proof of concept

Testing a simple flow, with four nodes.

* Inject  
* Function  
* HTTP Request  
* Debug  

![](https://github.com/jonathancraddock/Cachet-Config/blob/03fb5a2aa5753cfdca7f78c206ac8e2c6cac4747/screencap/nodered-test-put.png)

The **function** can be used to set the URL, the two required headers, and the data object should be written to msg.payload.

```javascript
msg.url = "https://cachet.example.com/api/v1/components/1";

msg.payload = {};
msg.payload = {"status":1};

msg.headers = {};
msg.headers['Content-Type'] = 'application/json';
msg.headers['X-Cachet-Token'] = 'API-KEY-HERE';

return msg;
```

Set the **HTTP Request** node to `PUT` and to return a `Parsed JSON Object`. In this example, the `status` of component 1 becomes set to `operational`.

-----

## References

### A Working Installation Guide

* https://websiteforstudents.com/install-cachet-status-platform-on-ubuntu-16-04-18-04-with-apache2-mariadb-and-php-7-2/  

### CachetHQ

* https://github.com/CachetHQ/Cachet 
* https://docs.cachethq.io/docs/installing-cachet  

### API

* https://oipwg.github.io/cachetapi/CachetAPI.html (node.js API reference)  

### Let's Encrypt

* https://certbot.eff.org/lets-encrypt/ubuntubionic-apache  
