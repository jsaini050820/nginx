# Nginx Full Setup on AWS EC2 (Amazon Linux 2)

Complete production-style Nginx setup on AWS EC2 including:

-   Static website hosting
-   Reverse proxy
-   Backend applications
-   Load balancing
-   Gzip compression
-   Rate limiting
-   Custom error pages
-   HTTPS with free SSL

------------------------------------------------------------------------

# Architecture

    Internet
       │
       ▼
    Nginx (EC2 Server)
       │
       ├── Static Website → /usr/share/nginx/html
       │
       ├── /app1 → Flask App (Port 5000)
       │
       ├── /app2 → Flask App (Port 5001)
       │
       └── /api → Load Balanced Backend

------------------------------------------------------------------------

# Prerequisites

-   AWS account
-   EC2 instance running Amazon Linux 2
-   Security Group allowing:

  Port   Purpose
  ------ ---------
  22     SSH
  80     HTTP
  443    HTTPS

------------------------------------------------------------------------

# 1. Launch EC2 Instance

Launch an instance using Amazon Linux 2.

Connect via SSH:

``` bash
ssh ec2-user@YOUR_PUBLIC_IP
```

------------------------------------------------------------------------

# 2. Update Server

``` bash
sudo yum update -y
```

------------------------------------------------------------------------

# 3. Install Nginx

Enable repository

``` bash
sudo amazon-linux-extras enable nginx1
```

Install Nginx

``` bash
sudo yum install nginx -y
```

Start service

``` bash
sudo systemctl start nginx
```

Enable auto start

``` bash
sudo systemctl enable nginx
```

Check version

``` bash
nginx -v
```

------------------------------------------------------------------------

# 4. Test Nginx

Open browser:

    http://EC2_PUBLIC_IP

If the Nginx welcome page appears, installation succeeded.

------------------------------------------------------------------------

# 5. Nginx Directory Structure

Main configuration

    /etc/nginx/nginx.conf

Site configuration

    /etc/nginx/conf.d/

Default site file

    /etc/nginx/conf.d/default.conf

Static website directory

    /usr/share/nginx/html

------------------------------------------------------------------------

# 6. Create Static Website

Navigate to web root

``` bash
cd /usr/share/nginx/html
```

Remove default page

``` bash
sudo rm index.html
```

Create new page

``` bash
sudo nano index.html
```

Example HTML

``` html
<!DOCTYPE html>
<html>
<head>
<title>Nginx Demo</title>
</head>

<body>

<h1>Welcome to My Nginx Server</h1>
<p>This website is hosted on Amazon Linux 2 EC2.</p>

</body>
</html>
```

------------------------------------------------------------------------

# 7. Install Python and Flask

``` bash
sudo yum install python3 -y
sudo yum install python3-pip -y
pip3 install flask
```

------------------------------------------------------------------------

# 8. Backend Application 1

Create file

``` bash
nano app1.py
```

``` python
from flask import Flask
app = Flask(__name__)

@app.route('/')
def home():
    return "Hello from App 1"

if __name__ == "__main__":
    app.run(host="0.0.0.0", port=5000)
```

Run app

``` bash
python3 app1.py
```

------------------------------------------------------------------------

# 9. Backend Application 2

Create file

``` bash
nano app2.py
```

``` python
from flask import Flask
app = Flask(__name__)

@app.route('/')
def home():
    return "Hello from App 2"

if __name__ == "__main__":
    app.run(host="0.0.0.0", port=5001)
```

Run app

``` bash
python3 app2.py
```

------------------------------------------------------------------------

# 10. Configure Reverse Proxy and Load Balancing

Edit configuration

``` bash
sudo nano /etc/nginx/conf.d/default.conf
```

Example configuration

``` nginx
upstream backend {
    server 127.0.0.1:5000;
    server 127.0.0.1:5001;
}

server {

    listen 80;
    server_name _;

    root /usr/share/nginx/html;
    index index.html;

    location /app1/ {
        proxy_pass http://127.0.0.1:5000/;

        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }

    location /app2/ {
        proxy_pass http://127.0.0.1:5001/;

        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }

    location /api/ {
        proxy_pass http://backend;
    }

    error_page 404 /404.html;

    location = /404.html {
        root /usr/share/nginx/html;
        internal;
    }

}
```

Test configuration

``` bash
sudo nginx -t
```

Reload server

``` bash
sudo systemctl reload nginx
```

------------------------------------------------------------------------

# 11. Enable Gzip Compression

Edit

``` bash
sudo nano /etc/nginx/nginx.conf
```

Inside the http block:

``` nginx
gzip on;
gzip_types text/plain text/css application/javascript application/json;
gzip_min_length 1000;
```

------------------------------------------------------------------------

# 12. Enable Rate Limiting

Inside http block

``` nginx
limit_req_zone $binary_remote_addr zone=mylimit:10m rate=1r/s;
```

Inside server block

``` nginx
location / {
    limit_req zone=mylimit burst=5;
}
```

------------------------------------------------------------------------

# 13. Custom 404 Page

``` bash
nano /usr/share/nginx/html/404.html
```

Example

``` html
<h1>404 Error</h1>
<p>Page not found</p>
```

------------------------------------------------------------------------

# 14. Enable HTTPS

Install Certbot

``` bash
sudo yum install epel-release -y
sudo yum install certbot python3-certbot-nginx -y
```

Generate SSL

``` bash
sudo certbot --nginx -d yourdomain.com
```

------------------------------------------------------------------------

# Final Result

You now have a complete Nginx setup with:

-   Static website hosting
-   Reverse proxy
-   Load balancing
-   Backend apps
-   Compression
-   Rate limiting
-   HTTPS

All running on a single EC2 instance.

------------------------------------------------------------------------

# License

MIT License
