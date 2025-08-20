# 🚀 VPS Setup Guide

This guide explains step-by-step how the `setup.sh` script configures your VPS for hosting backend, webhook, and React frontend services.

---

## 1. Configuration Variables

The script defines these key variables:

- **Backend Service**
  - App Name: `backend`
  - Port: `8080`
  - Domain: `backend.yourdomain.com`

- **Webhook Service**
  - App Name: `webhook-service`
  - Port: `8081`
  - Domain: `webhook.yourdomain.com`

- **React Frontend**
  - App Name: `react-app`
  - Port: `3001`
  - Domain: `app.yourdomain.com`

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

## 4. Configure Git SSH

```bash
mkdir -p ~/.ssh
chmod 700 ~/.ssh
ssh-keygen -t rsa -b 4096 -C "your_email@example.com"
eval "$(ssh-agent -s)"
ssh-add ~/.ssh/id_rsa
cat ~/.ssh/id_rsa.pub
```

- Copy the output of the last command and add it to GitHub/GitLab/Bitbucket under **SSH keys**.  
- Test connection with:  
  ```bash
  ssh -T git@github.com
  ```

---

## 5. Install Node.js LTS + npm

```bash
curl -fsSL https://deb.nodesource.com/setup_18.x | sudo -E bash -
sudo apt install -y nodejs
```

Installs **Node.js v18 LTS** and npm.

---

## 6. Install PM2

```bash
sudo npm install -g pm2
pm2 startup systemd -u $USER --hp $HOME
```

PM2 manages Node.js applications, keeping them running in the background.

---

## 7. Install Nginx

```bash
sudo apt install nginx -y
sudo systemctl enable nginx
sudo systemctl start nginx
```

Installs and starts the Nginx web server.

---

## 8. Run Backend with PM2

```bash
cd /path/to/backend
npm install
pm2 start npm --name backend -- run start
```

- The backend runs on **port 8080**.
- Managed by **PM2**.

---

## 9. Run Webhook with PM2

```bash
cd /path/to/webhook
npm install
pm2 start npm --name webhook-service -- run start
```

- The webhook service runs on **port 8081**.
- Managed by **PM2**.

---

## 10. Run React Frontend with PM2 (Port 3001)

```bash
cd /path/to/react-app
npm install
npm run build
sudo npm install -g serve
pm2 start serve --name react-app -- -s build -l 3001
```

- The React frontend runs on **port 3001**.
- Managed by **PM2**.

---

## 11. Configure Nginx for Backend

Creates `/etc/nginx/sites-available/backend`:

```nginx
server {
    listen 80;
    server_name backend.yourdomain.com;

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

Enable it:

```bash
sudo ln -s /etc/nginx/sites-available/backend /etc/nginx/sites-enabled/
```

---

## 12. Configure Nginx for Webhook

Creates `/etc/nginx/sites-available/webhook-service`:

```nginx
server {
    listen 80;
    server_name webhook.yourdomain.com;

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

Enable it:

```bash
sudo ln -s /etc/nginx/sites-available/webhook-service /etc/nginx/sites-enabled/
```

---

## 13. Configure Nginx for React Frontend

Creates `/etc/nginx/sites-available/react-app`:

```nginx
server {
    listen 80;
    server_name app.yourdomain.com;

    location / {
        proxy_pass http://localhost:3001/;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
    }
}
```

Enable it:

```bash
sudo ln -s /etc/nginx/sites-available/react-app /etc/nginx/sites-enabled/
```

---

## 14. Test & Reload Nginx

```bash
sudo nginx -t
sudo systemctl reload nginx
sudo systemctl restart nginx
```

---

## 15. Install Certbot & Enable HTTPS

```bash
sudo apt install -y certbot python3-certbot-nginx
sudo certbot --nginx -d backend.yourdomain.com -d webhook.yourdomain.com -d app.yourdomain.com
```

Requests SSL certificates for all domains with auto-redirect.

---

## 16. Setup SSL Auto-Renew

```bash
0 3 * * * /usr/bin/certbot renew --quiet && systemctl reload nginx
```

Added to cron to renew daily at 3 AM.

---

## 17. Enable PM2 Persistence

```bash
pm2 save
pm2 startup systemd -u $USER --hp $HOME
```

Ensures apps auto-start after reboot.

---

## ✅ Completion

- Git SSH is configured.
- Backend, Webhook, and React apps run via PM2.
- Nginx reverse-proxies each service to its domain.
- SSL certificates auto-renew.

---
