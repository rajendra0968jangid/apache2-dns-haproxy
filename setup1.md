# Apache Reverse Proxy with SSL using Certbot (Ubuntu 24.04)

## Project Overview

In this project, you will:

- Install Apache Web Server
- Create two websites running on different ports
- Configure Apache Virtual Hosts
- Configure Apache as a Reverse Proxy
- Secure the website using Let's Encrypt SSL (Certbot)

---

# Prerequisites

- Ubuntu 24.04 Server
- Domain Name: **hello.aicodecenter.com**
- DNS A Record pointing to your server's Public IP
- Open Ports:
  - 80 (HTTP)
  - 443 (HTTPS)

---

# Step 1: Domain Name

```
hello.aicodecenter.com
```

Verify that your domain points to your server.

---

# Step 2: Update Ubuntu

```bash
sudo apt update
```

---

# Step 3: Install Apache

```bash
sudo apt install apache2 -y
```

Check Apache Status

```bash
systemctl status apache2
```

Expected Output

```
Active: active (running)
```

---

# Step 4: Enable Required Apache Modules

Enable Proxy Modules

```bash
sudo a2enmod proxy
sudo a2enmod proxy_http
sudo a2enmod ssl
sudo a2enmod rewrite
```

Restart Apache

```bash
sudo systemctl restart apache2
```

---

# Step 5: Create Two Websites

Create Website Directories

```bash
sudo mkdir -p /var/www/host1
sudo mkdir -p /var/www/host2
```

### Website 1

```bash
sudo nano /var/www/host1/index.html
```

```html
<h1>Site 1 - Port 8081</h1>
```

### Website 2

```bash
sudo nano /var/www/host2/index.html
```

```html
<h1>Site 2 - Port 8082</h1>
```

---

# Step 6: Verify DNS

Check DNS Resolution

```bash
dig hello.aicodecenter.com
```

The returned IP should match your server's Public IP.

---

# Step 7: Configure Apache Ports

Edit Apache Ports Configuration

```bash
sudo nano /etc/apache2/ports.conf
```

Add

```apache
Listen 8081
Listen 8082
```

Save and Exit.

---

# Step 8: Configure Virtual Hosts

## Host 1 Configuration

Create Configuration File

```bash
sudo nano /etc/apache2/sites-available/host1.conf
```

```apache
<VirtualHost *:8081>

    DocumentRoot /var/www/host1

    <Directory /var/www/host1>
        AllowOverride All
        Require all granted
    </Directory>

</VirtualHost>
```

---

## Host 2 Configuration

```bash
sudo nano /etc/apache2/sites-available/host2.conf
```

```apache
<VirtualHost *:8082>

    DocumentRoot /var/www/host2

    <Directory /var/www/host2>
        AllowOverride All
        Require all granted
    </Directory>

</VirtualHost>
```

---

## Enable Virtual Hosts

```bash
sudo a2ensite host1.conf
sudo a2ensite host2.conf
```

Restart Apache

```bash
sudo systemctl restart apache2
```

---

# Step 9: Configure Apache Reverse Proxy

Create Reverse Proxy Configuration

```bash
sudo nano /etc/apache2/sites-available/hello.conf
```

```apache
<VirtualHost *:80>

    ServerName hello.aicodecenter.com

    ProxyPreserveHost On

    ProxyPass        /site1 http://127.0.0.1:8081/
    ProxyPassReverse /site1 http://127.0.0.1:8081/

    ProxyPass        /site2 http://127.0.0.1:8082/
    ProxyPassReverse /site2 http://127.0.0.1:8082/

    ErrorLog ${APACHE_LOG_DIR}/hello-error.log
    CustomLog ${APACHE_LOG_DIR}/hello-access.log combined

</VirtualHost>
```

Enable the Site

```bash
sudo a2ensite hello.conf
```

Reload Apache

```bash
sudo systemctl reload apache2
```

---

# Step 10: Install SSL using Certbot

Install Certbot

```bash
sudo apt install certbot python3-certbot-apache -y
```

Generate SSL Certificate

```bash
sudo certbot --apache -d hello.aicodecenter.com
```

Verify SSL

```bash
sudo systemctl status certbot.timer
```

Renew Certificate Test

```bash
sudo certbot renew --dry-run
```

---

# Step 11: Launch the Websites

Website 1

```
http://hello.aicodecenter.com/site1
```

Website 2

```
http://hello.aicodecenter.com/site2
```

After SSL Installation

```
https://hello.aicodecenter.com/site1
```

```
https://hello.aicodecenter.com/site2
```

---

# Verify Apache Configuration

Check Configuration Syntax

```bash
sudo apache2ctl configtest
```

Expected Output

```
Syntax OK
```

Restart Apache

```bash
sudo systemctl restart apache2
```

---

# Useful Commands

Restart Apache

```bash
sudo systemctl restart apache2
```

Reload Apache

```bash
sudo systemctl reload apache2
```

Check Apache Status

```bash
systemctl status apache2
```

List Enabled Sites

```bash
ls /etc/apache2/sites-enabled/
```

List Listening Ports

```bash
sudo ss -tulpn | grep apache2
```

View Apache Error Logs

```bash
sudo tail -f /var/log/apache2/error.log
```

View Reverse Proxy Logs

```bash
sudo tail -f /var/log/apache2/hello-error.log
```

---

# Project Architecture

```
                   Internet
                       │
                       │
          hello.aicodecenter.com
                       │
                 Port 80 / 443
                       │
                Apache Reverse Proxy
                       │
         ┌─────────────┴─────────────┐
         │                           │
         │                           │
      /site1                     /site2
         │                           │
     localhost                  localhost
       :8081                       :8082
         │                           │
     Apache Host1               Apache Host2
         │                           │
 /var/www/host1              /var/www/host2
```

---

# Project Summary

This project demonstrates:

- Apache Installation
- Apache Virtual Hosts
- Multiple Websites on Different Ports
- Apache Reverse Proxy Configuration
- Domain Name Integration
- SSL Certificate using Certbot
- HTTPS Deployment
- Reverse Proxy Routing (`/site1` and `/site2`)
