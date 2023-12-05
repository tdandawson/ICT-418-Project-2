# Installing The Apache Web Server 

I ran an update as it is standard practice.
```
sudo dnf update
```

I then ran the following command to install apache on LinuxRocky8:
```
sudo dnf install httpd
```

## Check the status and verion of the apache package
```
sudo httpd -v
```
## Start, enable, and check the status of the HTTPD service

Start command:
```
sudo systemctl start httpd
```

Enable command:
```
sudo systemctl enable httpd
```

Status command:
```
sudo systemctl status httpd
```

## Enable firewallD service
```
sudo firewall-cmd --permanent --add-port=80/tcp
sudo firewall-cmd --permanent --add-port=443/tcp
sudo firewall-cmd --reload
```

## Create Web Page
```
cd /var/www/html/
sudo mv index.html index.html.original
sudo nano index.html
```

I then added the provided text from our notes into the index.html file:
```
<html>
<head>
<title>My first web page using Apache2</title>
</head>
<body>

<h1>Welcome</h1>

<p>Welcome to my web site. I created this site using the Apache2 HTTP server.</p>

</body>
</html>
```
