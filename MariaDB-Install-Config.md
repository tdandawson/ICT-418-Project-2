# Installing and Configuring MariaDB

```
sudo dnf module list mariadb
```

## Install MariaDB

```
sudo dnf install mariadb-server mariadb
```

After execting this command, I was met with the error message below:

**Error: **
 Problem 1: cannot install the best candidate for the job
  - nothing provides libc.so.6(GLIBC_2.34)(64bit) needed by MariaDB-server-10.6.16-1.el9.x86_64 from mariadb
  - nothing provides libstdc++.so.6(GLIBCXX_3.4.29)(64bit) needed by MariaDB-server-10.6.16-1.el9.x86_64 from mariadb
  - nothing provides libm.so.6(GLIBC_2.29)(64bit) needed by MariaDB-server-10.6.16-1.el9.x86_64 from mariadb
  - nothing provides libstdc++.so.6(GLIBCXX_3.4.26)(64bit) needed by MariaDB-server-10.6.16-1.el9.x86_64 from mariadb
  - nothing provides libcrypt.so.2()(64bit) needed by MariaDB-server-10.6.16-1.el9.x86_64 from mariadb
  - nothing provides libcrypt.so.2(XCRYPT_2.0)(64bit) needed by MariaDB-server-10.6.16-1.el9.x86_64 from mariadb
 Problem 2: cannot install the best candidate for the job
  - nothing provides libc.so.6(GLIBC_2.34)(64bit) needed by MariaDB-client-10.6.16-1.el9.x86_64 from mariadb
  - nothing provides libstdc++.so.6(GLIBCXX_3.4.29)(64bit) needed by MariaDB-client-10.6.16-1.el9.x86_64 from mariadb
  - nothing provides libm.so.6(GLIBC_2.29)(64bit) needed by MariaDB-client-10.6.16-1.el9.x86_64 from mariadb
  - nothing provides libstdc++.so.6(GLIBCXX_3.4.26)(64bit) needed by MariaDB-client-10.6.16-1.el9.x86_64 from mariadb
  - nothing provides libcrypt.so.2()(64bit) needed by MariaDB-client-10.6.16-1.el9.x86_64 from mariadb
  - nothing provides libcrypt.so.2(XCRYPT_2.0)(64bit) needed by MariaDB-client-10.6.16-1.el9.x86_64 from mariadb
(try to add '--skip-broken' to skip uninstallable packages or '--nobest' to use not only best candidate packages)**


After scanning through the error message, I went back through and added libraries like we did in our assignment.

```
sudo mkdir /mustafar
ls -ld /mustafar
which bash
sudo mkdir /mustafar/bin
sudo cp /usr/bin/bash /mustafar/bin
ldd /usr/bin/bash
```

I then made the lib64 directory that was shown from `ldd /usr/bin/bash`.

```
sudo mkdir /mustafar/lib64
cd /mustafar/lib64
ls
```
I then copied the libraries from the `/lib64/` directory to the current `/mustafar/lib64/` directory.

```
sudo cp /lib64/libtinfo.so.6 .
sudo cp /lib64/libdl.so.2 .
sudo cp /lib64/libc.so.6 .
sudo cp /lib64/ld-linux-x86-64.so.2 .
ls
```

Then, I tried to install MariaDB again:
```
sudo dnf install mariadb-server
```

This resulted in another smaller error message:

**Error: 
 Problem: cannot install the best candidate for the job
  - nothing provides libc.so.6(GLIBC_2.34)(64bit) needed by MariaDB-server-10.6.16-1.el9.x86_64 from mariadb
  - nothing provides libstdc++.so.6(GLIBCXX_3.4.29)(64bit) needed by MariaDB-server-10.6.16-1.el9.x86_64 from mariadb
  - nothing provides libm.so.6(GLIBC_2.29)(64bit) needed by MariaDB-server-10.6.16-1.el9.x86_64 from mariadb
  - nothing provides libstdc++.so.6(GLIBCXX_3.4.26)(64bit) needed by MariaDB-server-10.6.16-1.el9.x86_64 from mariadb
  - nothing provides libcrypt.so.2()(64bit) needed by MariaDB-server-10.6.16-1.el9.x86_64 from mariadb
  - nothing provides libcrypt.so.2(XCRYPT_2.0)(64bit) needed by MariaDB-server-10.6.16-1.el9.x86_64 from mariadb
(try to add '--skip-broken' to skip uninstallable packages or '--nobest' to use not only best candidate packages)**


This made me decide to check the libraries for `ls` and `cat`:
```
ldd /usr/bin/ls
ldd /usr/bin/cat
```

After making sure all of the `ls` and `cat` libraries were copied to the `/mustafar/lib64/` directory, I tried installing MariaDB again:
```
sudo dnf install mariadb-server
```

However, I kept running into the same error message which resulted in me installing with `--nobest`:
```
sudo dnf install mariadb-server --nobest
sudo dnf install mariadb --nobest
```
This finally worked and MariaDB was successfully installed.

## Check status, start, enable, and check status of MariaDB again.

Check Status of MariaDB:
`
sudo systemctl status mariadb
`

Start MariaDB:
`
sudo systemctl start mariadb
`

Enable MariaDB:
`
sudo systemctl enable mariadb
`

Check Status Again:
`
sudo systemctl status mariadb
`

## Secure MariaDB Server and go through steps.

I ran this post installation script to set up the MariaDB root password and perform some security checks:
```
sudo mysql_secure_installation
```

## Login and test database.
```
mysql -u root -p
show databases;
```

## Create and Set up Regular User Account.
```
create user 'webapp'@'localhost' identified by 'XXXXXXX';
```

## Create Practice Database.
```
create database linuxrockydb;
grant all privileges on linuxrockydb.* to 'webapp'@'localhost';
show databases;
\q
```

## Login as Regular User and Create Tables.
```
mysql -u webapp -p
show databases;
use linuxrockydb;
create table distributions
    -> (
    -> id int unsigned not null auto_increment,
    -> name varchar(150) not null,
    -> developer varchar(150) not null,
    -> founded date not null,
    -> primary key (id)
    -> );
```

Check tables:
```
show tables;
describe distributions;
```
I added to the distributions table like we did in our assignment:
```
insert into distributions (name, developer, founded) values
    -> ('Debian', 'The Debian Project', '1993-09-15'),
    -> ('Ubuntu', 'Canonical Ltd.', '2004-10-20'),
    -> ('Fedora', 'Fedora Project', '2003-11-06');

select * from distributions;
\q
```

## Install PHP and MySQL support
```
sudo dnf install php-mysql
```

I then restarted Apache and MariaDB:
```
sudo systemctl restart httpd
sudo systemctl restart mariadb
```

## Create PHP Scripts.
```
cd /var/www/html
sudo touch login.php
sudo chmod 640 login.php
ls -l
sudo nano login.php
```
I installed the additional php files you sent via email:
```
sudo dnf install -y php php-zip php-intl php-mysqlnd php-dom php-simplexml php-xml php-xmlreader php-curl php-exif php-ftp php-gd php-iconv php-json php-mbstring php-posix php-sockets php-tokenizer
sudo chown -R apache:apache login.php

ls -l login.php

sudo nano login.php
sudo nano distros.php
```

## Testing Syntax

I tested the syntax to make sure there were no errors and everything was working properly:
```
sudo php -f login.php
sudo php -f index.php
sudo php -f distros.php
```
