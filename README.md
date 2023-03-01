# Ubuntu-Apache2-Php-Mysql-Configuration

## remote VPS

```
ssh username@ip
sudo su 	# access root
```

```
ssh -i file.pem username@ip
sudo -s 	# access root
```

## Install Apache2 & PHP-FPM

```
apt-get update && apt-get upgrade -y
apt-get install apache2 libapache2-mod-fcgid -y
cd /usr/lib/python3/dist-packages
cp apt_pkg.cpython-*-x86_64-linux-gnu.so apt_pkg.so
apt-get install software-properties-common -y
add-apt-repository ppa:ondrej/php
apt-get update
apt-get install php7.4 php7.4-fpm php7.4-common php7.4-mysql php7.4-xml php7.4-xmlrpc php7.4-curl php7.4-gd php7.4-imagick php7.4-cli php7.4-dev php7.4-imap php7.4-mbstring php7.4-soap php7.4-zip php7.4-bcmath php7.4-ssh2 php7.4-intl -y
a2enmod mpm_event proxy_fcgi setenvif
a2dismod mpm_prefork
a2enconf php7.4-fpm
a2enmod http2
a2enmod headers
a2enmod rewrite
systemctl restart apache2
```

### 000-default.conf

```
nano /etc/apache2/sites-available/000-default.conf
```

and replace all code with

```
<VirtualHost *:80>
    #ServerName www.example.com

    ServerAdmin webmaster@localhost
    DocumentRoot /var/www/public

    RewriteEngine On
    RewriteCond %{HTTPS} off
    RewriteRule (.*) https://%{HTTP_HOST}%{REQUEST_URI}
</VirtualHost>

<VirtualHost *:443>
    #ServerName www.example.com

    ServerAdmin webmaster@localhost
    DocumentRoot /var/www/public

    Protocols h2 h2c http/1.1
    H2EarlyHints on
    Header always set Strict-Transport-Security "max-age=15552000; includeSubDomains"

    SSLEngine on
    SSLCertificateFile /path/to/certs/certificate.crt				# edit here
    SSLCertificateKeyFile /path/to/private/private.key			# edit here
    SSLCertificateChainFile /path/to/certs/ca_bundle.crt		# edit here

    <Directory /var/www/public>
        Options -Indexes +FollowSymLinks +MultiViews
        AllowOverride All
        Require all granted
    </Directory>

    <FilesMatch \.php$>
        SetHandler "proxy:unix:/var/run/php/php7.4-fpm.sock|fcgi://localhost"
    </FilesMatch>

    ErrorLog ${APACHE_LOG_DIR}/error.log
    CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
```

```
systemctl restart apache2
```

### Optimization PHP-FPM

```
nano /etc/php/7.4/fpm/php.ini
```

```
upload_max_filesize = 256M
post_max_size = 256M
memory_limit = 512M
max_execution_time = 600
max_input_vars = 3000
max_input_time = 1000
```

```
service php7.4-fpm restart
```

## Install MariaDB

```
apt-get install apt-transport-https curl
curl -o /etc/apt/trusted.gpg.d/mariadb_release_signing_key.asc 'https://mariadb.org/mariadb_release_signing_key.asc'
sh -c "echo 'deb https://mirrors.gigenet.com/mariadb/repo/10.10/ubuntu bionic main' >>/etc/apt/sources.list"
apt-get update && apt-get install mariadb-server -y
mysql_secure_installation
```

### Create database

```
mysql_upgrade -u root -p
create database DATABASE_NAME;
grant all privileges on DATABASE_NAME.* TO 'root'@'localhost' identified by 'YOUR_PASSWORD';
```

## Create DIR Public

```
cd /var/www/
chown -R www-data:www-data /var/www/
mkdir public
cd public
chown -R www-data:www-data /var/www/public/
nano phpinfo.php
```

edit phpinfo.php with

```
<?php
	phpinfo();
```

## Test Access

`ip/phpinfo.php`

`local ip`

`example: 127.0.0.1/phpinfo.php`

`public ip`

`example: 104.100.bla.bla/phpinfo.php`
