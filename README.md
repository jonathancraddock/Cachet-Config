# Some Notes on Cachet Config

See: [Cachet](https://github.com/cachethq/Cachet)

## Installing on Ubuntu 20 LTS

I don't see many people complaining about the installation instructions, so maybe it's just me, but I tried multiple times with the official instructions, various blog posts, and then tried the Docker version. Basically nothing worked as expected. However, the guide on https://websiteforstudents.com/ does appear to work ok, and I've summarised my steps below.

Started with a clean Ubuntu 20 LTS install. I created a single sudo user and ran the usual updates. Took a snapshot here in case I needed to revert the setup, again!

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

There is a variety of conflicting advice around PHP versions. I tried 7.2 and it appears to work.

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

## API Examples

Update status of a component:  
```bash
curl -X PUT -H "Content-Type: application/json" -H "X-Cachet-Token: API-KEY-HERE" -d '{"status":1}' "https://cachet.example.com/api/v1/components/1"
```
^- *in this example, set component 1 to status of 'operational'*

-----

## References

### A Working Installation Guide

* https://websiteforstudents.com/install-cachet-status-platform-on-ubuntu-16-04-18-04-with-apache2-mariadb-and-php-7-2/  

### CachetHQ

* https://github.com/CachetHQ/Cachet 
* https://docs.cachethq.io/docs/installing-cachet  

### Let's Encrypt

* https://certbot.eff.org/lets-encrypt/ubuntubionic-apache  
