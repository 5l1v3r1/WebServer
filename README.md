# Webserver LAMP - Ubuntu 14.04

* Software
    * [Apache](https://github.com/davidecesarano/config-webserver-lamp#apache)
    * [PHP](https://github.com/davidecesarano/config-webserver-lamp#php)
    * [MySQL](https://github.com/davidecesarano/config-webserver-lamp#mysql)
    * [PhpMyAdmin](https://github.com/davidecesarano/config-webserver-lamp#phpmyadmin)
    * [ACL](https://github.com/davidecesarano/config-webserver-lamp#acl)
    * [Postfix](https://github.com/davidecesarano/config-webserver-lamp#postfix)
    * [SFTP](https://github.com/davidecesarano/config-webserver-lamp#sftp)
    * [Git and Composer](https://github.com/davidecesarano/config-webserver-lamp#git-and-composer)
* Security
    * [SSL Certificates (not sure)](https://github.com/davidecesarano/config-webserver-lamp#ssl-certificates-not-sure)
    * [SSL Certificates from Let's Encrypt (sure)](https://github.com/davidecesarano/config-webserver-lamp#ssl-certificates-from-lets-encrypt-sure)
    * [Apache](https://github.com/davidecesarano/config-webserver-lamp#apache-1)
    * [PHP](https://github.com/davidecesarano/config-webserver-lamp#php-1)
    * [PhpMyAdmin](https://github.com/davidecesarano/config-webserver-lamp#phpmyadmin-1)
* Users and Domains
    * [Create User](https://github.com/davidecesarano/config-webserver-lamp#create-user)
    * [Create Folders](https://github.com/davidecesarano/config-webserver-lamp#create-folders)
    * [Create Virtual Host](https://github.com/davidecesarano/config-webserver-lamp#create-virtual-host)
    * [Create automatic Backup](#create-automatic-backup)

## Software

### Apache

Install Apache:
```
$ sudo apt-get update
$ sudo apt-get -y install apache2
```

Active rewrite, headers and ssl modules:
```
$ sudo a2enmod rewrite
$ sudo a2enmod headers
$ sudo a2enmod ssl
```

Restart Apache:
```
$ sudo service apache2 restart
```

### PHP

Install PHP and PHP modules:
```
$ sudo apt-get update
$ sudo apt-get -y install php5-mysql php5 libapache2-mod-php5 php5-mcrypt php5-cli php5-gd php5-curl php5-xcache
```

Open the *dir.conf* file:
```
$ sudo nano /etc/apache2/mods-enabled/dir.conf
```

Move the PHP index file above to the first position after the *DirectoryIndex* specification:
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

Open the *000-default.conf* file:
```
$ sudo nano /etc/apache2/sites-available/000-default.conf
```

Clear all and add:
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

Install MySQL and follow the wizard:
```
$ sudo apt-get update
$ sudo apt-get -y install mysql-server
```

### PhpMyAdmin

Install PhpMyAdmin, follow the wizard and enable php5-mcrypt:
```
$ sudo apt-get install phpmyadmin
$ sudo php5enmod mcrypt
```

Restart Apache:
```
$ sudo service apache2 restart
```

### ACL		
 		
 Install ACL:		
 ```		
 $ sudo apt-get install acl		
 ```

### Postfix

Install Postfix and follow the wizard:
```
$ sudo apt-get install mailutils
```

### SFTP

Add the sftp group:
```
$ sudo addgroup sftp
```

Open the *sshd_config* file:
```
$ sudo nano /etc/ssh/sshd_config
```

Find *Subsystem*, comment it and add new: 
```
#Subsystem sftp /usr/lib/openssh/sftp-server
Subsystem sftp internal-sftp
```

After *UsePAM yes* add:
```
Match Group sftp
ChrootDirectory %h/domains/
X11Forwarding no
AllowTcpForwarding no
ForceCommand internal-sftp
```

Restart SSH:
```
$ sudo service ssh restart
```

### Git and Composer

Install Git and Composer:
```
$ sudo apt-get -y install git
$ curl -s https://getcomposer.org/installer | php
$ sudo mv composer.phar /usr/local/bin/composer
```

## Security

### SSL Certificates (not sure)

Create ssl folder:
```
$ sudo mkdir /etc/apache2/ssl
```

Create key following the wizard:
```
$ sudo openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout /etc/apache2/ssl/apache.key -out /etc/apache2/ssl/apache.crt
```

### SSL Certificates from Let’s Encrypt (sure)

Download the certbot-auto Let’s Encrypt client to the /usr/local/sbin directory
```
$ cd /usr/local/sbin
$ sudo wget https://dl.eff.org/certbot-auto
```

Make the script executable by typing:
```
$ sudo chmod a+x /usr/local/sbin/certbot-auto
```

Run the certbot-auto command with:
```
$ certbot-auto --apache -d EXAMPLE.COM -d WWW.EXAMPLE.COM
```

Create a new job with:
```
$ sudo crontab -e
```

Include the following content:
```
30 2 * * 1 /usr/local/sbin/certbot-auto renew >> /var/log/le-renew.log
```

### Apache

Open *security.conf* file:
```
$ sudo nano /etc/apache2/conf-available/security.conf
```

Hide Apache version:
```
ServerSignature Off
ServerTokens Prod
```

Restart Apache:
```
$ sudo service apache2 restart
```

Open *apache2.conf* file:
```
sudo nano /etc/apache2/apache2.conf
```

Disable directory listing:
```
<Directory /var/www/html>
   Options -Indexes
</Directory>
```

Restrict access to directories with “Allow” and “Deny” options :
```
<Directory />
   Options None
   Order deny,allow
   Deny from all
</Directory>
```

### PHP

Open *php.ini* file:
```
$ sudo nano /etc/php5/apache2/php.ini 
```

Set max filesize for upload file:
```
post_max_size = 25M
upload_max_filesize = 25M
```

Do not expose PHP error messages:
```
display_errors = Off
```

Restrict PHP Information Leakage:
```
expose_php = Off
```

Limit PHP access to file system:
```
open_basedir = "/var/www/html/"
upload_tmp_dir = "/var/tmp/"
```

Restart Apache:
```
$ sudo service apache2 restart
```

### PhpMyAdmin

Open the config.inc.php file:
```
$ sudo nano /etc/phpmyadmin/config.inc.php
```

After *$cfg['Servers'][$i]['recent']* add:
```
$cfg['Servers'][$i]['hide_db'] = '^information_schema|mysql|performance_schema|phpmyadmin$';
```

After *$cfg['SaveDir']* add:
```
$cfg['ForceSSL'] = true;
```

Open the *apache.conf* file:
```
$ sudo nano /etc/phpmyadmin/apache.conf
```

Find */phpmyadmin* and replace with:
```
Alias /database/domains /usr/share/phpmyadmin
```

Restart Apache:
```
$ sudo service apache2 restart
```

## Users and Domains

### Create User

Create user, set home folder and add user to www-data and sftp groups:
```
$ useradd -m -d /var/www/USERNAME -s /bin/false -g www-data -G sftp USERNAME
$ passwd USERNAME
```

### Create Folders

Set root user and root group for home user folder:
```
$ sudo chown root:root /var/www/USERNAME
```

Create domains folder in home user folder:
```
$ mkdir /var/www/USERNAME/domains
```

Create domain name (EXAMPLE.COM) folder in domains folder:
```
$ mkdir /var/www/USERNAME/domains/EXAMPLE.COM
```

Create httpdocs, logs and backups folders in EXAMPLE.COM folder:
```
$ mkdir -p /var/www/USERNAME/domains/EXAMPLE.COM/{httpdocs,logs,backups}
```

Create index.php file in httpdocs folder and set permissions:
```
$ sudo touch /var/www/USERNAME/domains/EXAMPLE.COM/httpdocs/index.php
$ echo "<?php echo '<h1>It\'s Works!</h1>'; ?>" > /var/www/USERNAME/domains/EXAMPLE.COM/httpdocs/index.php
```

Set permissions to httpdocs folder:

```
$ chown -R USERNAME:www-data /var/www/USERNAME/domains/EXAMPLE.COM/httpdocs
```

Set the default user (USERNAME), default group (www-data), 775 permissions for folders and 664 permissions for files with ACL:

```
$ setfacl -R -m u:USERNAME:rwx /var/www/USERNAME/domains/EXAMPLE.COM/httpdocs
$ setfacl -Rd -m u:USERNAME:rwx /var/www/USERNAME/domains/EXAMPLE.COM/httpdocs
$ setfacl -R -m g:www-data:rwx /var/www/USERNAME/domains/EXAMPLE.COM/httpdocs
$ setfacl -Rd -m g:www-data:rwx /var/www/USERNAME/domains/EXAMPLE.COM/httpdocs
```

### Create Virtual Host

Create *example.com.conf* file:
```
$ sudo nano /etc/apache2/sites-available/example.com.conf
```

Add:
```
<VirtualHost *:80>
    ServerAdmin webmaster@localhost
    ServerName example.com
    ServerAlias www.example.com
    DocumentRoot /var/www/USERNAME/domains/EXAMPLE.COM/httpdocs
    ErrorLog /var/www/USERNAME/domains/EXAMPLE.COM/logs/error.log
    CustomLog /var/www/USERNAME/domains/EXAMPLE.COM/logs/access.log combined
    <Directory /var/www/USERNAME/domains/EXAMPLE.COM/httpdocs>
        Options FollowSymLinks MultiViews
        AllowOverride All
        Order allow,deny
        allow from al
    </Directory>
</VirtualHost>  
```

Enable virtual host:
```
$ sudo a2ensite example.com.conf
```

Restart Apache:
```
$ sudo service apache2 restart
```

### Create automatic Backup

In root folder create backup folder:

```
$ mkdir -p /root/sh/backup
```

In backup folder create *example.com.sh* file:

```
$ sudo nano /root/sh/backup/example.com.sh
```

To create a weekly automatic backup add:

```
mysqldump -u USER -pPASSWORD DATABASENAME | gzip > /var/www/USERNAME/domains/EXAMPLE.COM/backups/database.sql.gz
tar czf /var/www/USERNAME/domains/EXAMPLE.COM/backups/httpdocs.tar.gz -C / var/www/USERNAME/domains/EXAMPLE.COM/httpdocs
find /var/www/USERNAME/domains/EXAMPLE.COM/backups/httpdocs* -mtime +7 -exec rm {} \;
find /var/www/USERNAME/domains/EXAMPLE.COM/backups/database* -mtime +7 -exec rm {} \;
```

Set permissions:

```
$ sudo chmod +x /root/sh/backup/example.com.sh
```

Create a new job with:

```
$ crontab -l
```

To create an automatic backup every Saturday at 4:00 am:

```
00 04 * * Sat /root/sh/backup/example.com.sh
```