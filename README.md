# Webserver LAMP - Ubuntu 14.04

## Software

### Apache

```
$ sudo apt-get update
$ sudo apt-get -y install apache2
```

Active...
```
$ sudo a2enmod rewrite
$ sudo a2enmod headers
$ sudo a2enmod ssl
$ sudo service apache2 restart
```

### PHP

```
$ sudo apt-get update
$ sudo apt-get -y install php5-mysql php5 libapache2-mod-php5 php5-mcrypt php5-cli php5-gd php5-curl php5-xcache
```

Open the dir.conf file:
```
$ sudo nano /etc/apache2/mods-enabled/dir.conf
```

Move the PHP index file above to the first position after the DirectoryIndex specification:
```
<IfModule mod_dir.c>
    DirectoryIndex index.php index.html index.cgi index.pl index.xhtml index.htm
</IfModule>
```

Create root directory and index.php file:
```
$ sudo mkdir /var/www/html/root
$ sudo touch /var/www/html/root/index.php
$ echo "<?php echo '<h1>It\'s Works!</h1>'; ?>" > /var/www/html/root/index.php
```

```
$ sudo nano /etc/apache2/sites-available/000-default.conf
```

```
<VirtualHost *:80>
    ServerAdmin webmaster@localhost
    DocumentRoot /var/www/html/root
    ErrorLog ${APACHE_LOG_DIR}/error.log
    CustomLog ${APACHE_LOG_DIR}/access.log combined
    <Directory /var/www/html/root>
        Options -Indexes -Includes
        Order allow,deny
        Allow from all
    </Directory>
</VirtualHost>  
```

Restart Apache:
```
$ sudo service apache2 restart
```

### MySQL

```
$ sudo apt-get update
$ sudo apt-get -y install mysql-server
```

### PhpMyAdmin

```
$ sudo apt-get install phpmyadmin
$ sudo php5enmod mcrypt
$ sudo service apache2 restart
```

### ACL

```
$ sudo apt-get install acl
```

### Mailutils

```
$ sudo apt-get install mailutils
```

### SFTP

```
$ sudo addgroup sftp
$ sudo nano /etc/ssh/sshd_config
```

```
#Subsystem sftp /usr/lib/openssh/sftp-server
Subsystem sftp internal-sftp
```

```
Match Group sftp
ChrootDirectory %h/domains/
X11Forwarding no
AllowTcpForwarding no
ForceCommand internal-sftp
```

```
$ sudo service ssh restart
```

### Git and Composer

```
$ sudo apt-get -y install git
$ curl -s https://getcomposer.org/installer | php
$ sudo mv composer.phar /usr/local/bin/composer
```

## Security

### Apache

ServerSignature Off
ServerTokens Prod

### PHP

```
$ sudo nano /etc/apach2/conf-available/security.conf 
```

```
    ServerTokens Prod
    ServerSignature Off
    Header always append X-Frame-Options SAMEORIGIN
    Header set X-XSS-Protection: "1; mode=block"
    Header edit Set-Cookie ^(.*)$ $1;HttpOnly;Secure
    Timeout 60
```

### PhpMyAdmin

```
$ sudo nano /etc/phpmyadmin/config.inc.php
    $cfg['Servers'][$i]['hide_db'] = '^information_schema|mysql|performance_schema|phpmyadmin$';
    $cfg['ForceSSL'] = true;
$ sudo nano /etc/phpmyadmin/apache.conf
    Alias /database/domains /usr/share/phpmyadmin
$ sudo service apache2 restart
```

## Users and Domains

### Create User

```
$ useradd -m -d /var/www/html/USER -s /bin/false -g www-data -G sftp USER
$ passwd USER
```

### Create Directory

```
$ sudo chown root:root /var/www/html/USER
$ mkdir /var/www/html/USER/domains
$ mkdir /var/www/html/USER/domains/EXAMPLE.COM
$ mkdir -p /var/www/html/$UTENTE/domains/EXAMPLE.COM/{httpdocs,logs,backups}
$ setfacl -R -m u:USER:rwx /var/www/html/USER/domains/EXAMPLE.COM/httpdocs
$ setfacl -Rd -m u:USER:rwx /var/www/html/USER/domains/EXAMPLE.COM/httpdocs
$ setfacl -R -m g:www-data:rwx /var/www/html/USER/domains/EXAMPLE.COM/httpdocs
$ setfacl -Rd -m g:www-data:rwx /var/www/html/USER/domains/EXAMPLE.COM/httpdocs
$ sudo touch /var/www/html/USER/domains/EXAMPLE.COM/httpdocs/index.php
$ echo "<?php echo '<h1>It\'s Works!</h1>'; ?>" > /var/www/html/USER/domains/EXAMPLE.COM/httpdocs/index.php
$ sudo chown USER:www-data /var/www/html/USER/domains/EXAMPLE.COM/httpdocs/index.php
```

### Create Virtual Host

### Create Backup
