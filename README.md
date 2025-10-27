🖥️ VPS SETUP

Complete guide to set up a VPS with Node.js, Nginx, MongoDB, PM2, and SSL.

STEP 1 — Connect to VPS
<pre><code id="example-code"> ssh root@your_ip_adress </code></pre>

Then enter your password.

STEP 2 — Update Packages
<pre><code id="example-code"> sudo apt update </code></pre> <pre><code id="example-code"> sudo apt upgrade </code></pre>
STEP 3 — Enable & Configure UFW (Firewall)
<pre><code id="example-code"> sudo ufw enable </code></pre> <pre><code id="example-code"> sudo ufw allow 22 </code></pre>

Check allowed ports:

<pre><code id="example-code"> sudo ufw status </code></pre>
STEP 4 — Install NVM & Node.js

Install NVM:

<pre><code id="example-code"> curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.40.1/install.sh | bash </code></pre>

Check version:

<pre><code id="example-code"> nvm -v </code></pre>

Install Node.js LTS:

<pre><code id="example-code"> nvm install --lts </code></pre>

(Optional: install a specific version)

<pre><code id="example-code"> nvm install 18.17.1 (optional - for specific version) </code></pre>

Restart terminal and check:

<pre><code id="example-code"> node -v </code></pre> <pre><code id="example-code"> npm -v </code></pre>

⚠️ If versions don’t show, close and reopen the terminal.

STEP 5 — GitHub SSH Connection

Check existing SSH keys:

<pre><code id="example-code"> ls -al ~/.ssh </code></pre>

Generate new SSH key:

<pre><code id="example-code"> ssh-keygen -t rsa -b 4096 -C "github@smtech24.com" </code></pre>

Start SSH agent:

<pre><code id="example-code"> eval "$(ssh-agent -s)" </code></pre>

Copy SSH public key:

<pre><code id="example-code"> cat ~/.ssh/id_rsa.pub </code></pre>

Then add it to GitHub → SSH and GPG Keys → New SSH Key.

Test connection:

<pre><code id="example-code"> ssh -T git@github.com </code></pre>
STEP 6 — Setup Nginx

Allow ports:

<pre><code id="example-code"> sudo ufw allow 80/tcp sudo ufw allow 3000 sudo ufw allow 443/tcp sudo ufw reload </code></pre>

Install and enable Nginx:

<pre><code id="example-code"> sudo apt install nginx -y sudo systemctl status nginx sudo systemctl start nginx sudo systemctl enable nginx sudo ufw allow 'Nginx Full' sudo ufw enable </code></pre>
STEP 7 — Configure Nginx for Next.js (Frontend)
<pre><code id="example-code"> sudo nano /etc/nginx/sites-available/yourdomain.com </code></pre>

Paste:

<pre><code id="example-code"> server { listen 80; server_name yourdomain.com www.yourdomain.com; location / { proxy_pass http://localhost:3000; # Next.js runs on port 3000 proxy_http_version 1.1; proxy_set_header Upgrade $http_upgrade; proxy_set_header Connection 'upgrade'; proxy_set_header Host $host; proxy_cache_bypass $http_upgrade; } } </code></pre>

Enable configuration:

<pre><code id="example-code"> sudo ln -s /etc/nginx/sites-available/yourdomain.com /etc/nginx/sites-enabled sudo nginx -t sudo systemctl reload nginx </code></pre>
STEP 8 — Configure Nginx for Backend
<pre><code id="example-code"> sudo nano /etc/nginx/sites-available/yourdomain.com </code></pre>

Paste:

<pre><code id="example-code"> server{ listen 80; server_name api.myfinancialtrading.com; location / { proxy_pass http://localhost:3000; proxy_set_header Host $host; proxy_set_header X-Forwarded-Host $host; proxy_set_header X-Forwarded-Proto $scheme; proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for; } } </code></pre>

Save & exit:

<pre><code id="example-code"> ctrl + o (Write Out) Enter ctrl + x (Exit) </code></pre>

Enable configuration:

<pre><code id="example-code"> sudo ln -s /etc/nginx/sites-available/api.yourdomain.com /etc/nginx/sites-enabled sudo nginx -t sudo systemctl reload nginx </code></pre>
STEP 9 — Install Project & Setup
<pre><code id="example-code"> cd /var/www git clone <git repository using ssh> cd project-name npm install npm run build npm run dev # Test if running properly </code></pre>

Stop local dev:

<pre><code id="example-code"> CTRL + Z </code></pre>

If port already in use:

Change PORT in .env

Commit & push:

<pre><code id="example-code"> git add . git commit -m "Changed port" git push </code></pre>

Pull, install, build:

<pre><code id="example-code"> git pull npm install npm run build </code></pre>

Allow new port:

<pre><code id="example-code"> sudo ufw allow <PORT> </code></pre>
STEP 10 — Run with PM2
<pre><code id="example-code"> npm install -g pm2 pm2 --version pm2 start npm --name "project-frontend" -- start # Next.js frontend pm2 start dist/server.js --name project-backend # Backend pm2 list pm2 startup pm2 save pm2 restart all </code></pre>

Check logs:

<pre><code id="example-code"> pm2 logs </code></pre>

If error:

<pre><code id="example-code"> sudo lsof -i :<PORT> sudo kill -9 PID pm2 logs </code></pre>

Test deployment:

<pre><code id="example-code"> http://<SERVER_IP>:<PORT> </code></pre>

Stop/Delete server:

<pre><code id="example-code"> pm2 stop nextjs-app pm2 delete nextjs-app </code></pre>
STEP 11 — Install SSL
<pre><code id="example-code"> sudo apt install certbot python3-certbot-nginx -y sudo certbot --nginx -d yourdomain.com -d www.yourdomain.com </code></pre>
STEP 12 — Install MongoDB with Prisma ORM

Ubuntu 22.04 → MongoDB 8.0
Ubuntu 22.00 → MongoDB 6.0

<pre><code id="example-code"> sudo apt-get install gnupg curl curl -fsSL https://www.mongodb.org/static/pgp/server-8.0.asc | \ sudo gpg -o /usr/share/keyrings/mongodb-server-8.0.gpg --dearmor echo "deb [ arch=amd64,arm64 signed-by=/usr/share/keyrings/mongodb-server-8.0.gpg ] https://repo.mongodb.org/apt/ubuntu jammy/mongodb-org/8.0 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-8.0.list # If not working, use: echo "deb [ arch=amd64,arm64 signed-by=/usr/share/keyrings/mongodb-server-8.0.gpg ] https://repo.mongodb.org/apt/ubuntu noble/mongodb-org/8.0 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-8.0.list sudo apt-get update sudo apt-get install -y mongodb-org sudo systemctl start mongod sudo systemctl daemon-reload sudo systemctl status mongod sudo systemctl restart mongod mongosh </code></pre>
Prisma Replica Setup
<pre><code id="example-code"> sudo nano /etc/mongod.conf replication: replSetName: "rs0" sudo systemctl restart mongod mongosh rs.initiate() rs.status() ctrl + z sudo systemctl status mongod sudo ufw allow 27017 sudo ufw reload DATABASE_URL="mongodb://127.0.0.1:27017/lunatix?replicaSet=rs0" </code></pre>
