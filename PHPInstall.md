# Installing and Configuring PHP

## Restart Apache

```
sudo systemctl restart httpd
```

## Install EPEL repository on Rocky Linux

```
sudo dnf install epel-release
```

## Enable remi repository
```
sudo dnf install dnf-utils http://rpms.remirepo.net/enterprise/remi-release-8.rpm
```

## Reset default php module and enbale the latest Remi-PHP module (Remi-9.0)
```
sudo dnf module reset php
sudo dnf module enable php:remi-8.0
```

## Install PHP 8.0 and common extensions
```
sudo dnf install php php-cli php-curl php-mysqlnd php-gd php-opcache php-zip php-intl
```

## Confirm
```
php -v
```

## Check Install
```
cd /var/www/html
sudo nano info.php
```

### I did not have nano installed so I had to install with:
```
sudo dnf install nano -y
```

## Retry
```
sudo nano info.php
```

### I added the provided text from our notes
```
<?php
phpinfo();
?>
```

