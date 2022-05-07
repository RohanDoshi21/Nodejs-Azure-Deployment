> Steps to deploy a Node.js app to Azure using PM2, NGINX as a reverse proxy

## 1. Sign up for Azure


## 2. Create a Virtual Machine
 select Ubuntu 20.04<br>
 select connect using password<br>
 setup an user and password<br>
 after the virtual machine has been you will get its public ip adress
 
## 3. Connect to VM
 ```
ssh username@ipaddress
```
enter the prompted password

## 4. Install Node/NPM
```
curl -sL https://deb.nodesource.com/setup_12.x | sudo -E bash -

sudo apt install nodejs

node --version
```

## 5. Clone your project from Github
```
git clone yourproject.git
```

### 6. Install dependencies and test app
```
cd yourproject
npm install
npm start
# stop app
ctrl+C
```

## 7. Setup PM2 process manager to keep your app running
```
sudo npm i pm2 -g
pm2 start app

# Other pm2 commands
pm2 show app
pm2 status
pm2 restart app
pm2 stop app
pm2 logs (Show log stream)
pm2 flush (Clear logs)

# To make sure app starts when reboot
pm2 startup ubuntu
```
### You should now be able to access your app using your IP and port. Now we want to setup a firewall blocking that port and setup NGINX as a reverse proxy so we can access it directly using port 80 (http)

## 8. Setup ufw firewall
```
sudo ufw enable
sudo ufw status
sudo ufw allow ssh (Port 22)
sudo ufw allow http (Port 80)
sudo ufw allow https (Port 443)
```

## 9. Install NGINX and configure
```
sudo apt install nginx

sudo nano /etc/nginx/sites-available/default
```
Add the following to the location part of the server block
```
    server_name _;

    location / {
        proxy_pass http://localhost:8080; #port your app runs on
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
    }
```
```
# Check NGINX config
sudo nginx -t

# Restart NGINX
sudo service nginx restart
```

### You should now be able to visit your IP with no port (port 80) and see your app.

## To kill a process on a port 
```
kill $(lsof -t -i:8080)
```
