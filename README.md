🖥️ VPS SETUP GUIDE

A complete step-by-step guide to set up your VPS for deploying Next.js, Node.js, and MongoDB projects using Nginx, PM2, and Certbot SSL.

📚 Table of Contents

STEP 1 — Connect to VPS

STEP 2 — Update Packages

STEP 3 — Enable & Configure UFW

STEP 4 — Install NVM & Node.js

STEP 5 — GitHub SSH Setup

STEP 6 — Setup Nginx

STEP 7 — Configure Nginx (Frontend & Backend)

STEP 8 — Install Project & Setup

STEP 9 — Install SSL

STEP 10 — Install MongoDB with Prisma ORM

⚙️ STEP 1 — Connect to VPS
ssh root@your_ip_address


Then enter your password.

🔄 STEP 2 — Update Packages
sudo apt update

sudo apt upgrade

🔐 STEP 3 — Enable & Configure UFW (Firewall)
sudo ufw enable

sudo ufw allow 22


✅ Check allowed ports:

sudo ufw status

🟢 STEP 4 — Install NVM & Node.js

Install NVM:

curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.40.1/install.sh | bash


Check NVM version:

nvm -v


Install Node.js:

nvm install --lts


(Optional — install a specific version)

nvm install 18.17.1


Restart your terminal, then verify:

node -v

npm -v


⚠️ If versions are not showing, close and reopen the terminal, then try again.

🔗 STEP 5 — GitHub SSH Setup

Check for existing SSH keys:

ls -al ~/.ssh


Generate new SSH key:

ssh-keygen -t rsa -b 4096 -C "github@smtech24.com"


Start SSH agent:

eval "$(ssh-agent -s)"


View your SSH public key:

cat ~/.ssh/id_rsa.pub


Copy the output and add it to your GitHub → Settings → SSH and GPG Keys → New SSH Key.
Give it a project name and paste the key.

Test connection:

ssh -T git@github.com

🌐 STEP 6 — Setup Nginx

Allow Nginx and common ports:

sudo ufw allow 80/tcp
sudo ufw allow 3000
sudo ufw allow 443/tcp
sudo ufw reload


Install and enable Nginx:

sudo apt install nginx -y
sudo systemctl status nginx
sudo systemctl start nginx
sudo systemctl enable nginx
sudo ufw allow 'Nginx Full'
sudo ufw enable

⚛️ STEP 7 — Configure Nginx (Frontend & Backend)
🧩 For Next.js Frontend
sudo nano /etc/nginx/sites-available/yourdomain.com


Paste this configuration:

server {
  listen 80;
  server_name yourdomain.com www.yourdomain.com;

  location / {
      proxy_pass http://localhost:3000; # Next.js runs on port 3000
      proxy_http_version 1.1;
      proxy_set_header Upgrade $http_upgrade;
      proxy_set_header Connection 'upgrade';
      proxy_set_header Host $host;
      proxy_cache_bypass $http_upgrade;
  }
}


Then run:

sudo ln -s /etc/nginx/sites-available/yourdomain.com /etc/nginx/sites-enabled
sudo nginx -t
sudo systemctl reload nginx

⚙️ For Backend Server
sudo nano /etc/nginx/sites-available/yourdomain.com


Paste this:

server {
  listen 80;
  server_name api.myfinancialtrading.com;

  location / {
    proxy_pass  http://localhost:3000;
    proxy_set_header Host $host;
    proxy_set_header X-Forwarded-Host $host;
    proxy_set_header X-Forwarded-Proto $scheme;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
  }
}


Save and exit:

CTRL + O  → Enter
CTRL + X  → Exit


Enable configuration:

sudo ln -s /etc/nginx/sites-available/api.yourdomain.com /etc/nginx/sites-enabled
sudo nginx -t
sudo systemctl reload nginx

🚀 STEP 8 — Install Project & Setup
cd /var/www
git clone <git repository using ssh>
npm install
npm run build
npm run dev   # Test if running properly


Stop the local dev run:

CTRL + Z

🧰 If Port Error Occurs:

Open .env file → change the PORT value → then:

git add .
git commit -m "Changed port"
git push
git pull
npm install
npm run build


Allow new port:

sudo ufw allow <PORT>

🔁 Run with PM2
npm install -g pm2
pm2 --version
pm2 start npm --name "project-frontend" -- start   # For Next.js frontend
pm2 start dist/server.js --name project-backend    # For backend
pm2 list
pm2 startup
pm2 save
pm2 restart all


Check logs:

pm2 logs

⚠️ If Any Error:
sudo lsof -i :<PORT>
sudo kill -9 <PID>
pm2 logs


Test deployment:

http://<SERVER_IP>:<PORT>


Stop or delete app:

pm2 stop nextjs-app
pm2 delete nextjs-app

🔒 STEP 9 — Install SSL (HTTPS)
sudo apt install certbot python3-certbot-nginx -y
sudo certbot --nginx -d yourdomain.com -d www.yourdomain.com

🗄️ STEP 10 — Install MongoDB with Prisma ORM

For Ubuntu 22.04, use MongoDB 8.0
For Ubuntu 22.00, use MongoDB 6.0

Installation Steps
sudo apt-get install gnupg curl

curl -fsSL https://www.mongodb.org/static/pgp/server-8.0.asc | \
sudo gpg -o /usr/share/keyrings/mongodb-server-8.0.gpg --dearmor

echo "deb [ arch=amd64,arm64 signed-by=/usr/share/keyrings/mongodb-server-8.0.gpg ] https://repo.mongodb.org/apt/ubuntu jammy/mongodb-org/8.0 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-8.0.list


If not working, use this:

echo "deb [ arch=amd64,arm64 signed-by=/usr/share/keyrings/mongodb-server-8.0.gpg ] https://repo.mongodb.org/apt/ubuntu noble/mongodb-org/8.0 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-8.0.list

sudo apt-get update
sudo apt-get install -y mongodb-org
sudo systemctl start mongod
sudo systemctl daemon-reload
sudo systemctl status mongod
sudo systemctl restart mongod
mongosh

⚙️ Prisma Replica Setup
sudo nano /etc/mongod.conf


Add:

replication:
  replSetName: "rs0"


Restart and configure:

sudo systemctl restart mongod
mongosh
rs.initiate()
rs.status()
CTRL + Z   # Exit mongosh
sudo systemctl status mongod
sudo ufw allow 27017
sudo ufw reload


Set your .env variable:

DATABASE_URL="mongodb://127.0.0.1:27017/lunatix?replicaSet=rs0"
