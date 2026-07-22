# HAProxy Load Balancer & Apache Multi-Host Setup with Let's Encrypt SSL

This guide provides a step-by-step walkthrough for configuring an **Apache Web Server** hosting multiple virtual hosts behind an **HAProxy Load Balancer** with **Let's Encrypt (Certbot) SSL termination**.

---

## 🏗 System Architecture Overview

* **Frontend Proxy:** HAProxy (Ports `80` and `443`)
* **Backend Web Server:** Apache2
  * Host 1: Listening on `127.0.0.1:8081`
  * Host 2: Listening on `127.0.0.1:8082`
* **Load Balancing Algorithm:** Round Robin
* **SSL Certificate:** Let's Encrypt ACME Challenge via Port `8888`

---

## 🛠 Step 1: Install Required Packages

Update package repositories and install Apache2 along with HAProxy:

```bash
sudo apt update
sudo apt install -y apache2 haproxy
```

---

## ⚙️ Step 2: Configure Apache Ports

Modify the Apache ports configuration so that Apache listens on custom internal ports (`8081` and `8082`) instead of default port `80`.

Open `/etc/apache2/ports.conf`:

```bash
sudo nano /etc/apache2/ports.conf
```

Replace or update the file content to:

```apache
Listen 8081
Listen 8082
```

---

## 📁 Step 3: Create Apache Web Host Directories & Files

Create root directories for both virtual hosts and add sample `index.html` files.

```bash
sudo mkdir -p /var/www/host1 /var/www/host2
```

### Create `index.html` for Apache Host 1:

```bash
sudo nano /var/www/host1/index.html
```

Paste the following HTML content:

```html
<!DOCTYPE html>
<html>
  <head>
    <title>Host 1</title>
  </head>
  <body style="background-color: #e6f3ff; font-family: sans-serif; text-align: center; padding-top: 50px;">
    <h1 style="color: #0066cc;">Served by Apache Host 1</h1>
    <p>Running on Port 8081</p>
  </body>
</html>
```

### Create `index.html` for Apache Host 2:

```bash
sudo nano /var/www/host2/index.html
```

Paste the following HTML content:

```html
<!DOCTYPE html>
<html>
  <head>
    <title>Host 2</title>
  </head>
  <body style="background-color: #e6ffe6; font-family: sans-serif; text-align: center; padding-top: 50px;">
    <h1 style="color: #009933;">Served by Apache Host 2</h1>
    <p>Running on Port 8082</p>
  </body>
</html>
```

---

## 🌐 Step 4: Configure Apache Virtual Hosts

Create a unified Virtual Host configuration for both backends.

```bash
sudo nano /etc/apache2/sites-available/multi-host.conf
```

Paste the following configuration:

```apache
<VirtualHost *:8081>
    DocumentRoot /var/www/host1
    DirectoryIndex index.html
    <Directory /var/www/host1>
        AllowOverride All
        Require all granted
    </Directory>
</VirtualHost>

<VirtualHost *:8082>
    DocumentRoot /var/www/host2
    DirectoryIndex index.html
    <Directory /var/www/host2>
        AllowOverride All
        Require all granted
    </Directory>
</VirtualHost>
```

Disable the default site, enable `multi-host.conf`, and restart Apache:

```bash
sudo a2dissite 000-default.conf
sudo a2ensite multi-host.conf
sudo systemctl restart apache2
```

---

## ⚖️ Step 5: Configure HAProxy (Load Balancer & ACME Route)

Backup the existing HAProxy configuration and create a new one.

```bash
sudo mv /etc/haproxy/haproxy.cfg /etc/haproxy/haproxy.cfg.bak
sudo nano /etc/haproxy/haproxy.cfg
```

Paste the initial HAProxy configuration:

```haproxy
global
    log /dev/log local0
    log /dev/log local1 notice
    chroot /var/lib/haproxy
    user haproxy
    group haproxy
    daemon

defaults
    log     global
    mode    http
    option  httplog
    option  dontlognull
    timeout connect 5000ms
    timeout client  50000ms
    timeout server  50000ms

# HTTP Frontend (Port 80)
frontend http_front
    bind *:80
    
    # Path reserved for Certbot SSL challenge
    acl letsencrypt-acl path_beg /.well-known/acme-challenge/
    use_backend letsencrypt-backend if letsencrypt-acl
    
    # Default traffic routes to Apache backend
    default_backend apache_backend

# Apache Backend (Round Robin Load Balancer)
backend apache_backend
    balance roundrobin
    option httpchk GET /index.html
    server host1 127.0.0.1:8081 check
    server host2 127.0.0.1:8082 check

# Backend reserved for SSL Renewal
backend letsencrypt-backend
    server letsencrypt 127.0.0.1:8888
```

Restart HAProxy to apply changes:

```bash
sudo systemctl restart haproxy
```

---

## 🔒 Step 6: Enable HTTPS (SSL Certificate Setup)

### 1. Install Certbot

```bash
sudo apt install -y certbot
```

### 2. Request SSL Certificate

Request an SSL certificate for `hello.aicodecenter.com` using port `8888` (which routes through HAProxy's ACME backend):

```bash
sudo certbot certonly --standalone --preferred-challenges http --http-01-port 8888 -d hello.aicodecenter.com
```

### 3. Combine Certificate & Private Key for HAProxy

HAProxy requires the certificate chain and private key to be combined into a single `.pem` file:

```bash
sudo mkdir -p /etc/haproxy/certs
sudo bash -c 'cat /etc/letsencrypt/live/hello.aicodecenter.com/fullchain.pem /etc/letsencrypt/live/hello.aicodecenter.com/privkey.pem > /etc/haproxy/certs/hello.aicodecenter.com.pem'
sudo chmod 600 /etc/haproxy/certs/hello.aicodecenter.com.pem
```

### 4. Update HAProxy Configuration for SSL & HTTP Redirection

Edit `/etc/haproxy/haproxy.cfg`:

```bash
sudo nano /etc/haproxy/haproxy.cfg
```

Update `http_front` to redirect all HTTP traffic to HTTPS, and add the `https_front` frontend:

```haproxy
frontend http_front
    bind *:80
    acl letsencrypt-acl path_beg /.well-known/acme-challenge/
    use_backend letsencrypt-backend if letsencrypt-acl
    
    # Redirect all regular HTTP traffic to HTTPS
    redirect scheme https code 301 if !letsencrypt-acl

frontend https_front
    bind *:443 ssl crt /etc/haproxy/certs/hello.aicodecenter.com.pem
    default_backend apache_backend
```

### 5. Restart HAProxy

Apply the HTTPS configuration:

```bash
sudo systemctl restart haproxy
```

---

## ✅ Verification

1. **HTTP Redirect:** Open `http://hello.aicodecenter.com` in your browser. It should automatically redirect to `https://hello.aicodecenter.com`.
2. **Load Balancing:** Refresh `https://hello.aicodecenter.com` multiple times. You should see responses alternating between **Apache Host 1** (Port 8081) and **Apache Host 2** (Port 8082).
