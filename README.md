# 🚀 VPS Setup Script

This repository contains a Bash script (`setup.sh`) to automatically configure a VPS with **Node.js**, **PM2**, **Nginx**, and **SSL certificates** for backend and webhook services.

---

## 📌 Features

- Installs required packages: Git, Node.js (LTS), PM2, and Nginx
- Configures **Nginx reverse proxy** for backend and webhook services
- Automatically generates and installs **SSL certificates** via Certbot (Let's Encrypt)
- Configures **HTTPS with auto-redirect**
- Adds **daily cron job** for SSL auto-renewal
- Ensures **PM2 persistence** across server reboots

---

## ⚙️ Configuration

Edit the script variables at the top of `setup.sh` if needed:

```bash
# Backend Service
BACKEND_APP_NAME="backend"
BACKEND_PORT=8080
BACKEND_DOMAIN="backend.homeslygroup.com"

# Webhook Service
WEBHOOK_APP_NAME="webhook-service"
WEBHOOK_PORT=8081
WEBHOOK_DOMAIN="cloudbed.homeslygroup.com"
```

---

## 📖 Setup Steps

### 1. Update System
```bash
sudo apt update && sudo apt upgrade -y
```

### 2. Install Git
```bash
sudo apt install git -y
```

### 3. Install Node.js LTS + npm
```bash
curl -fsSL https://deb.nodesource.com/setup_lts.x | sudo -E bash -
sudo apt install -y nodejs
```

### 4. Install PM2
```bash
sudo npm install -g pm2
pm2 startup systemd -u $USER --hp $HOME
```

### 5. Install Nginx
```bash
sudo apt install nginx -y
sudo systemctl enable nginx
sudo systemctl start nginx
```

### 6. Configure Nginx (Backend + Webhook)

Example backend configuration:
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

Webhook configuration is similar, but on port `8081`.

### 7. Test & Reload Nginx
```bash
sudo nginx -t
sudo systemctl reload nginx
sudo systemctl restart nginx
```

### 8. Install Certbot & Enable HTTPS
```bash
sudo apt install -y certbot python3-certbot-nginx
sudo certbot --nginx -d backend.homeslygroup.com --redirect -m admin@backend.homeslygroup.com --agree-tos -n
sudo certbot --nginx -d cloudbed.homeslygroup.com --redirect -m admin@backend.homeslygroup.com --agree-tos -n
```

### 9. Setup Auto-Renew SSL
```bash
0 3 * * * /usr/bin/certbot renew --quiet && systemctl reload nginx
```

### 10. Enable PM2 Persistence
```bash
pm2 startup systemd -u $USER --hp $HOME
pm2 save
```

---

## ▶️ How to Run the Script

1. Clone the repository or copy the script to your VPS.
2. Make the script executable:
   ```bash
   chmod +x setup.sh
   ```
3. Run the script:
   ```bash
   ./setup.sh
   ```

This will automatically set up your VPS with all configurations.

---

## ✅ Completion

- Nginx reverse proxy is configured for **backend** and **webhook**
- SSL certificates auto-renew daily at 3 AM
- PM2 ensures apps auto-start after reboot

---

## 📄 License
This script is open-source and free to use.
