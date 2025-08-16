# 🚀 VPS Setup Guide

This guide explains step-by-step how the `setup.sh` script configures your VPS for hosting backend and webhook services.

---

## 1. Configuration Variables

The script defines these key variables:

- **Backend Service**
  - App Name: `backend`
  - Port: `8080`
  - Domain: `backend.homeslygroup.com`

- **Webhook Service**
  - App Name: `webhook-service`
  - Port: `8081`
  - Domain: `cloudbed.homeslygroup.com`

---

## 2. System Update

```bash
sudo apt update && sudo apt upgrade -y
```

Updates the VPS system packages to the latest versions.

---

## 3. Install Git

```bash
sudo apt install git -y
```

Installs Git for version control.

---

## 4. Install Node.js LTS + npm

```bash
curl -fsSL https://deb.nodesource.com/setup_lts.x | sudo -E bash -
sudo apt install -y nodejs
```

Installs Node.js (LTS version) and npm.

---

## 5. Install PM2

```bash
sudo npm install -g pm2
pm2 startup systemd -u $USER --hp $HOME
```

PM2 manages Node.js applications, keeping them running in the background.

---

## 6. Install Nginx

```bash
sudo apt install nginx -y
sudo systemctl enable nginx
sudo systemctl start nginx
```

Installs and starts the Nginx web server.

---

## 7. Configure Nginx for Backend

Creates `/etc/nginx/sites-available/backend`:

```nginx
server {
    listen 80;
    server_name backend.homeslygroup.com;

    location / {
        proxy_pass http://localhost:8080/;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
    }
}
```

Then symlinks it to `sites-enabled/`.

---

## 8. Configure Nginx for Webhook

Creates `/etc/nginx/sites-available/webhook-service`:

```nginx
server {
    listen 80;
    server_name cloudbed.homeslygroup.com;

    location / {
        proxy_pass http://localhost:8081/;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
    }
}
```

Then symlinks it to `sites-enabled/`.

---

## 9. Test & Reload Nginx

```bash
sudo nginx -t
sudo systemctl reload nginx
sudo systemctl restart nginx
```

Ensures configuration is valid and reloads Nginx.

---

## 10. Install Certbot & Enable HTTPS

```bash
sudo apt install -y certbot python3-certbot-nginx
```

Requests SSL certificates for both domains with auto-redirect.

---

## 11. Setup SSL Auto-Renew

A cron job is added to auto-renew SSL daily at 3 AM:

```bash
0 3 * * * /usr/bin/certbot renew --quiet && systemctl reload nginx
```

---

## 12. Enable PM2 Persistence

```bash
pm2 startup systemd -u $USER --hp $HOME
pm2 save
```

Ensures backend and webhook apps auto-start after reboot.

---

## ✅ Completion

- SSL certificates will auto-renew daily at 3 AM.
- Backend & Webhook services will auto-start after reboot.

---
