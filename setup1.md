# Apache Reverse Proxy Project on Ubuntu 24.04

## Project Overview
This project demonstrates how to host multiple websites using **Apache Reverse Proxy** on a single domain.

**Domain:** `hello.aicodecenter.com`

---

# Step 1: Domain Name

Use the following domain:

```text
hello.aicodecenter.com
```

---

# Step 2: Server

Operating System:

```text
Ubuntu 24.04 LTS
```

---

# Step 3: Install Apache2

Update the package repository and install Apache.

```bash
sudo apt update
sudo apt install apache2 -y
```

Verify Apache is running:

```bash
systemctl status apache2
```

Expected output:

```
Active: active (running)
```

---

# Step 4: Enable Required Apache Modules

Enable the modules required for reverse proxy.

```bash
sudo a2enmod proxy
sudo a2enmod proxy_http
sudo a2enmod ssl
sudo a2enmod rewrite
```

Restart Apache.

```bash
sudo systemctl restart apache2
```

---

# Step 5: Create Website Directories

Create directories for two websites.

```bash
sudo mkdir -p /var/www/host1
sudo mkdir -p /var/www/host2
```

### Website 1

Create the HTML page.

```bash
sudo nano /var/www/host1/index.html
```

Add:

```html
<h1>Site 1 - Port 8081</h1>
```

Save the file.

---

### Website 2

Create another HTML page.

```bash
sudo nano /var/www/host2/index.html
```

Add:

```html
<h1>Site 2 - Port 8082</h1>
```

Save the file.

---

# Step 6: Verify DNS

Confirm that your domain points to the server.

```bash
dig hello.aicodecenter.com
```

Verify that the returned IP address matches your server's public IP.

---

# Step 7: Configure Apache Reverse Proxy

Create a new virtual host.

```bash
sudo nano /etc/apache2/sites-available/hello.conf
```

Paste the following configuration:

```apache
<VirtualHost *:80>

    ServerName hello.aicodecenter.com

    ProxyPreserveHost On

    ProxyPass /site1 http://127.0.0.1:8081/
    ProxyPassReverse /site1 http://127.0.0.1:8081/

    ProxyPass /site2 http://127.0.0.1:8082/
    ProxyPassReverse /site2 http://127.0.0.1:8082/

    ErrorLog ${APACHE_LOG_DIR}/hello-error.log
    CustomLog ${APACHE_LOG_DIR}/hello-access.log combined

</VirtualHost>
```

Save the file.

---

# Step 8: Enable the Virtual Host

Enable the new site.

```bash
sudo a2ensite hello.conf
```

Reload Apache.

```bash
sudo systemctl reload apache2
```

---

# Step 9: Launch the Websites

Open the following URLs in your browser.

### Website 1

```
http://hello.aicodecenter.com/site1
```

Expected Output:

```
Site 1 - Port 8081
```

---

### Website 2

```
http://hello.aicodecenter.com/site2
```

Expected Output:

```
Site 2 - Port 8082
```

---

# Project Architecture

```
                    Internet
                        │
                        │
         hello.aicodecenter.com
                        │
                Apache (Port 80)
                        │
        ┌───────────────┴───────────────┐
        │                               │
   /site1                          /site2
        │                               │
        ▼                               ▼
 localhost:8081                  localhost:8082
        │                               │
   Website 1                       Website 2
```

---

# Commands Summary

```bash
sudo apt update
sudo apt install apache2 -y

sudo a2enmod proxy
sudo a2enmod proxy_http
sudo a2enmod ssl
sudo a2enmod rewrite

sudo systemctl restart apache2

sudo mkdir -p /var/www/host1
sudo mkdir -p /var/www/host2

dig hello.aicodecenter.com

sudo nano /etc/apache2/sites-available/hello.conf

sudo a2ensite hello.conf
sudo systemctl reload apache2
```

---

# Final Result

Visit:

- http://hello.aicodecenter.com/site1
- http://hello.aicodecenter.com/site2

Both websites are served through a single Apache server using **Reverse Proxy** while routing requests to different backend services running on ports **8081** and **8082**.
