# Webserver LAMP - Ubuntu 18.04

* Software
    * [Apache](https://github.com/davidecesarano/config-webserver-lamp#apache)
    * [PHP 7](https://github.com/davidecesarano/config-webserver-lamp#php-7)
    * [MySQL](https://github.com/davidecesarano/config-webserver-lamp#mysql)
    * [PhpMyAdmin](https://github.com/davidecesarano/config-webserver-lamp#phpmyadmin)
    * [ACL](https://github.com/davidecesarano/config-webserver-lamp#acl)
    * [Postfix](https://github.com/davidecesarano/config-webserver-lamp#postfix)
    * [SFTP](https://github.com/davidecesarano/config-webserver-lamp#sftp)
    * [Git and Composer](https://github.com/davidecesarano/config-webserver-lamp#git-and-composer)
    * [SSL Certificates from Let's Encrypt](https://github.com/davidecesarano/config-webserver-lamp#ssl-certificates-from-lets-encrypt)
* Users and Virtual Hosts
    * [Create User](https://github.com/davidecesarano/config-webserver-lamp#create-user)
    * [Create User's Folders](https://github.com/davidecesarano/config-webserver-lamp#create-user-s-folders)
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
$ sudo a2enmod rewrite headers ssl
```

Restart Apache:
```
$ sudo service apache2 restart
```

### PHP 7

Install PHP and PHP modules:
```
$ sudo apt-get update
$ sudo apt-get install php libapache2-mod-php
```

Create index.php file:
```
$ sudo touch /var/www/html/index.php
$ echo "<?php echo '<h1>It\'s Works!</h1>'; ?>" > /var/www/html/index.php
```

Open the *000-default.conf* file:
```
$ sudo nano /etc/apache2/sites-available/000-default.conf
```

Clear all and add:
```
<VirtualHost *:80>
    ServerAdmin webmaster@localhost
    DocumentRoot /var/www/html
    ErrorLog ${APACHE_LOG_DIR}/error.log
    CustomLog ${APACHE_LOG_DIR}/access.log combined
    <Directory /var/www/html>
        Options -Indexes +FollowSymLinks +MultiViews
        AllowOverride All
        Require all granted
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

Install PhpMyAdmin, follow the wizard and enable mbstring:
```
$ sudo apt-get update
$ sudo apt install phpmyadmin php-mbstring php-gettext
$ sudo phpenmod mbstring
```

Restart Apache:
```
$ sudo service apache2 restart
```

Create dedicated user to access phpMyAdmin:
```
$ mysql -u root -p
mysql> CREATE USER 'YOURUSERNAME'@'localhost' IDENTIFIED BY 'YOURPASSWORD';
mysql> GRANT ALL PRIVILEGES ON *.* TO 'YOURUSERNAME'@'localhost' WITH GRANT OPTION;
mysql> exit
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

Next, add the lines below at the end of the file:
```
Match Group sftp
X11Forwarding no
AllowTcpForwarding no
ChrootDirectory %h/html/vhosts
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

### SSL Certificates from Let’s Encrypt

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

## Users and Virtual Hosts

### Create User

Create user, set home folder and add user to www-data and sftp groups:
```
$ useradd -m -d /var/www/USERNAME -g www-data -G sftp USERNAME
$ passwd USERNAME
```

### Create User's Folders

Set root user and root group for home user folder:
```
$ sudo chown root:root /var/www/USERNAME
```

Create html folder in home user folder e vhosts folder in html folder:
```
$ mkdir /var/www/USERNAME/html
$ mkdir /var/www/USERNAME/html/vhosts
```

Set permissions to vhosts folder:

```
$ chown -R USERNAME:www-data /var/www/USERNAME/html/vhosts
```

Set the default user (USERNAME), default group (www-data), 775 permissions for folders and 664 permissions for files with ACL:

```
$ setfacl -R -m u:USERNAME:rwx /var/www/USERNAME/html/vhosts/*
$ setfacl -Rd -m u:USERNAME:rwx /var/www/USERNAME/html/vhosts/*
$ setfacl -R -m g:www-data:rwx /var/www/USERNAME/html/vhosts/*
$ setfacl -Rd -m g:www-data:rwx /var/www/USERNAME/html/vhosts/*
```

With your SFTP software you can to create virtual host's folders in vhosts folder. For example:
```
vhosts
    - example.com
        - httpdocs
            - index.php
            - ...
        - logs
    - example2.com
        - ...
```

### Create Virtual Host

Create *example.com.conf* file:
```
$ sudo nano /etc/apache2/sites-enabled/example.com.conf
```

Add:
```
<VirtualHost *:80>
    ServerName example.com
    DocumentRoot /var/www/USERNAME/domains/EXAMPLE.COM/httpdocs
    
    ErrorLog /var/www/USERNAME/domains/EXAMPLE.COM/logs/error.log
    CustomLog /var/www/USERNAME/domains/EXAMPLE.COM/logs/access.log combined
    
    <Directory /var/www/USERNAME/domains/EXAMPLE.COM/httpdocs>
        Options -Indexes +FollowSymLinks +MultiViews
        AllowOverride All
        Require all granted
    </Directory>
    
</VirtualHost> 
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
