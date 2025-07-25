# Remix App Deployment Guide

## Prerequisites
Before starting the deployment, ensure you have the necessary dependencies installed:

### 1. Install Nginx
```sh
sudo apt update
sudo apt install nginx -y
```

### 2. Install Node.js using NVM
Install NVM (Node Version Manager):
```sh
curl -fsSL https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.3/install.sh | bash
source ~/.bashrc
```
Verify installation:
```sh
nvm --version
```
Install Node.js (latest LTS version):
```sh
nvm install --lts
nvm use --lts
```
Verify Node.js and npm installation:
```sh
node -v
npm -v
```

### 3. Install MySQL Server
```sh
sudo apt install mysql-server -y
```

### 3.1 Install Postgres Server

```sh
sudo apt install postgresql postgresql-contrib
sudo systemctl start postgresql
sudo systemctl enable postgresql
```

### 4 Check Postgres status
```sh
sudo systemctl status postgresql
```

### 5. Switch to postgres user
```sh
sudo -i -u postgres
```

### 6. Access PostgreSQL shell
```sh
psql
```

### 7. Change the password
```sh
\password postgres
```

### Exit psql
```sh
\q
```
or press Ctrl + D

## Step 1: Create Nginx Configuration
Navigate to Nginx Sites-Available Directory:
```sh
cd /etc/nginx/sites-available
```

Create a Configuration File:
```sh
sudo nano [your-domain]
```

Add the following configuration:
```nginx
server {
    listen 80;
    server_name [your-domain];

    location / {
        proxy_pass http://localhost:3000;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
    }
}
```
Save and exit (`CTRL + X`, then `Y`, then `Enter`).

## Step 2: Create Symbolic Link
```sh
sudo ln -s /etc/nginx/sites-available/[your-domain] /etc/nginx/sites-enabled/
```

Test Nginx configuration:
```sh
sudo nginx -t
```

Reload Nginx:
```sh
sudo systemctl restart nginx
```

## Step 3: Apply SSL Certificate
Install Certbot:
```sh
sudo apt-get install certbot python3-certbot-nginx -y
```

Apply SSL Certificate:
```sh
sudo certbot --nginx -d [your-domain]
```

Install SSL Certificate (if needed):
```sh
sudo certbot install --cert-name [your-domain]
```

Auto-renew SSL certificate:
```sh
sudo certbot renew --dry-run
```

## Step 4: Deploy Your Project
Clone Your Project:
```sh
cd /var/www/html/
git clone [your-repo-url]
```

Install Dependencies:
```sh
cd [your-project-directory]
npm install
```

Change the Database password

```sh
mysql -u root -p
```

```sh
press enter (if password not configured yet)
or type the password
```

update the pass if not

```sh
ALTER USER 'root'@'localhost' IDENTIFIED WITH caching_sha2_password BY '[YOUR PASSWORD]';
EXIT;
```

create an .env file

```sh
sudo nano .env
```

fill up these values in .env

```sh
DATABASE_URL=mysql://host:[YOUR PASSWORD]@localhost:3306/database  (if using mysql)
SHOPIFY_API_KEY=xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx     (get it from in partner dashboard app config mentioned as Client ID)
SHOPIFY_API_SECRET=xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx  (get it from in partner dashboard app config mentioned as Client secret)
SHOPIFY_APP_URL=http://app.fontmarket.com/        ([your-domain])
SCOPES=read_files,write_files,write_products        ([scopes needed])
```

Sync MySQL with Remix:
```sh
npm run setup
```

## Step 5: Install and Configure PM2
Install PM2:
```sh
npm install pm2 -g
```

Build Your Application:
```sh
SHOPIFY_API_KEY=[API_KEY] npm run build
```

Start Application with PM2:
```sh
pm2 start npm --name "[handle]" -- run start
```

## Final Step: Restart Nginx
```sh
sudo systemctl restart nginx
```

## Additional Commands
### Reset MySQL Root Password
```sh
sudo mysql
```
Inside MySQL shell:
```sql
ALTER USER 'root'@'localhost' IDENTIFIED WITH caching_sha2_password BY '[YOUR PASSWORD]';
FLUSH PRIVILEGES;
EXIT;
```

### Fix `.env` Not Loading Issue
```sh
rm -rf node_modules/
npm install
```

## Summary
1. Install Nginx, Node.js (via NVM), and MySQL.
2. Create and configure the Nginx reverse proxy.
3. Apply SSL with Certbot.
4. Deploy your Remix project.
5. Use PM2 for process management.
6. Update extension configurations.
7. Restart Nginx.

Your **Remix app** should now be deployed, secured with SSL, and running on **PM2** with **Nginx** as a reverse proxy.

