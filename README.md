# How I Deployed and Secured `seo.payroyal.online` with SSL on Ubuntu (AWS EC2)

This guide documents the step-by-step process I followed to deploy a static HTML website on an Ubuntu server (hosted on EC2), configure **Nginx**, and secure it with **HTTPS using Certbot and Let's Encrypt**.

---

## 1. Project Overview

**Goal:**

Deploy an Nginx-hosted site on Ubuntu using an EC2 instance and secure it with Let's Encrypt SSL.

**Stack Used:**

- Ubuntu 24.04 LTS (EC2)
- Nginx
- Certbot (Let's Encrypt)
- Domain: `seo.payroyal.online`

---

## 2. Initial Server Setup

```

# Update and install basic tools
sudo apt update && sudo apt upgrade -y
sudo apt install nginx -y

```

---

### 3.  Domain & DNS Configuration

1. Logged into domain registrar (e.g., Namecheap).
2. Added the following A record:
    - **Host:** SEO
    - **Value:** `<EC2 public IP>`

📝 *Propagation may take a few minutes.*

### **4. Download and Deploy the Website**

I used a free static template from TemplateMo.

```
wget <https://templatemo.com/download/templatemo_582_tale_seo_agency> -O tale_seo_agency.zip
unzip tale_seo_agency.zip -d tale_seo_agency
sudo mv tale_seo_agency/* /var/www/html/
sudo rm /var/www/html/index.nginx-debian.html 
```

### 5.Nginx Configuration

```

sudo nano /etc/nginx/sites-available/default

```

**Configured a basic server block:**

```
server {
    listen 80;
    server_name seo.payroyal.online;

    root /var/www/html;
    index index.html index.htm;

    location / {
        try_files $uri $uri/ =404;
    }
}

```
# Test and reload Nginx

```
sudo nginx -t
sudo systemctl reload nginx

```

---

### 6. Fixing DNS Resolution Issues (resolvectl)

When i dig and resolvectl and it  shows DNS errors ( SERVFAIL), i was able to trobleshoot the error by updating the DNS configuration using:

```
sudo nano /etc/systemd/resolved.conf
```

Then Modify the config:

```

[Resolve]
DNS=8.8.8.8 1.1.1.1
FallbackDNS=8.8.4.4 1.0.0.1
```

And Restart the resolver:

```

sudo systemctl restart systemd-resolved
```

Check if DNS is resolved:

```

dig seo.payroyal.online

```

### 7. Installing SSL with Certbot

i installed Install Certbot and the Nginx plugin:

```
sudo apt install certbot python3-certbot-nginx -y
```

Run Certbot for my domain:

```
sudo certbot --nginx -d seo.payroyal.online
```

**Certbot did the following:**

- Auto-configured Nginx
- Installed SSL certificate
- Enabled HTTP to HTTPS redirect
- Scheduled automatic renewal

### 7. Verifying HTTPS & Renewal**

✅ Confirmed green lock at: https://seo.payroyal.online

✅ Verified HTTP→HTTPS redirection

✅ Checked renewal status:

```
sudo systemctl list-timers | grep certbot
```

### 8. Final Notes

Tools like dig and resolvectl were helpful for DNS debugging.

The website is now live, secured with SSL, and auto-renewing via Certbot.

Website is now live and secure: https://seo.payroyal.online

