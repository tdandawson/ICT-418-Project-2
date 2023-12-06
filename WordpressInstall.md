# Install Wordpress

## Create Database

To start, I created a database to hold all the installation and post-installation files for wordpress:
```
sudo mysql -u root -p
CREATE DATABASE wordpress_db;
```

Then, I created the database user and assigned a password:
```
CREATE USER 'wordpress_user'@'localhost' IDENTIFIED BY 'XXXXXXX';
```

Then I granted all privileges to the database user on the Wordpress database:
```
GRANT ALL ON wordpress_db.* TO 'wordpress_user'@'localhost';
```

Save changes and exit:
```
FLUSH PRIVILEGES;
EXIT;
```

After getting an error trying to download wordpress, I had to install `wget`:
```
sudo dnf -y install wget
```

## Download Wordpress.

Now that I had `wget` installed, I used the following command to download Wordpress in Rocky Linux:
```
wget https://wordpress.org/latest.tar.gz -O wordpress.tar.gz
```
I then extracted the compressed file:
```
tar -xvf wordpress.tar.gz
```

After that, I copied the uncompressed wordpress directory to the webroot folder:
```
sudo cp -R wordpress /var/www/html/
```

## Set ownership.

I set the ownership of the wordpress directory to the apache user and group:
```
sudo chown -R apache:apache /var/www/html/wordpress
```

To be completely honest, I forget why I did this again but I executed the following command:
```
sudo dnf install php-curl php-xml php-imagick php-mbstring php-zip php-intl
```


I think I must've gotten lost or mixed up and restarted everything:
```
sudo systemctl restart httpd
sudo systemctl restart mariadb
sudo systemctl restart mysql
```

Something happened to where I reverted to following our notes again (I can't remember what exactly was happening):
```
sudo wget https://wordpress.org/latest.tar.gz
sudo tar -xzvf latest.tar.gz
```

I Created Another Database and User:
```
mysql -u root -p
create user 'wordpress_user'@'localhost' identified by 'XXXXXXXXX';
create database wordpress_db;
grant all privileges on wordpress.* to 'wordpress_user'@'localhost';
show databases;
\q
```

## Set up wp-config.php

Change to the wordpress directory:
```
cd /var/www/html/wordpress
```

Copy and rename the wp-config-sample.php file to wp-config.php:
```
sudo cp wp-config-sample.php wp-config.php
```

Edit the file and add WordPress database name, user name, and password in the fields for DB_NAME, DB_USER, and DB_PASSWORD:
```
sudo nano wp-config.php
```

## Change file ownership.
```
sudo chown -R apache:apache *
ls -l
```
