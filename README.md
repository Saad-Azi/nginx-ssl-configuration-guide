# Nginx Configuration Guide for Subdomains

This guide walks you through the process of setting up Nginx for a subdomain, including configuring SSL/TLS with Certbot for secure HTTPS access.

---

## Step 1: Install Nginx
Update the package list and install Nginx:

```bash
sudo apt update
sudo apt install nginx
```
## Step 2: Create a Configuration File for Your Subdomain
Navigate to the sites-available directory:

```bash
cd /etc/nginx/sites-available/
```
Create a new configuration file for your subdomain:

```bash
sudo nano your-subdomain.conf
```
Add the following configuration, adjusting the domain and backend service details accordingly:

```bash
server {
    listen 80;
    server_name your-subdomain.example.com;

    location / {
        proxy_pass http://localhost:8000;  # Adjust this to the correct address of your backend service
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }

    # Optional: Add SSL configuration if using HTTPS
}
```
Save and exit the file by pressing Ctrl + X, then Y, and Enter.

##  Step 3: Enable the Site Configuration
Create a symbolic link to enable your site:

```bash
sudo ln -s /etc/nginx/sites-available/your-subdomain.conf /etc/nginx/sites-enabled/
```

## Step 4: Test Nginx Configuration for Syntax Errors
Check for syntax errors in the Nginx configuration:

```bash
sudo nginx -t
```
## Step 5: Reload Nginx to Apply the Configuration
Reload Nginx to apply the new configuration:

```bash
sudo systemctl reload nginx
```
## Step 6: Adjust Firewall Settings
If you're using UFW (Uncomplicated Firewall), allow traffic for Nginx:

```bash
sudo ufw allow 'Nginx Full'
```
## Step 7: Install Certbot for SSL Configuration
Install Certbot and the Nginx plugin to manage SSL certificates:

```bash
sudo apt install certbot python3-certbot-nginx
```
## Step 8: Obtain and Install the SSL Certificate
Use Certbot to automatically configure SSL for your domain:

```bash
sudo certbot --nginx -d your-subdomain.example.com
```
Follow the prompts to complete the SSL installation. Certbot will automatically configure Nginx to use SSL.

## Step 9: Enable Automatic SSL Certificate Renewal
To ensure that your SSL certificates renew automatically, enable the Certbot timer:

```bash
sudo systemctl enable certbot.timer
```
## Step 10: Update Nginx Configuration for SSL
After installing the SSL certificate, update your Nginx configuration to redirect HTTP traffic to HTTPS and enable SSL:

Edit the Nginx configuration file:

```bash
sudo nano /etc/nginx/sites-available/your-subdomain.conf
```
Update the configuration to include the SSL settings:

```bash
server {
    listen 80;
    server_name your-subdomain.example.com;
    return 301 https://$host$request_uri;  # Redirect HTTP to HTTPS
}

server {
    listen 443 ssl;
    server_name your-subdomain.example.com;

    ssl_certificate /etc/letsencrypt/live/your-subdomain.example.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/your-subdomain.example.com/privkey.pem;
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers 'ECDHE-RSA-AES128-GCM-SHA256:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-RSA-AES128-SHA256:ECDHE-RSA-AES256-SHA384:DHE-RSA-AES128-GCM-SHA256:DHE-RSA-AES256-GCM-SHA384';
    ssl_prefer_server_ciphers on;

    location / {
        proxy_pass http://localhost:8000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```
Save the file (Ctrl + X, then Y, and Enter).

## Step 11: Test and Reload Nginx
Test the updated Nginx configuration:

```bash
sudo nginx -t
```
Reload Nginx to apply the SSL changes:

```bash
sudo systemctl reload nginx
```
## Step 12: Verify SSL/TLS Configuration
Finally, verify that SSL is working by visiting your subdomain using HTTPS (https://your-subdomain.example.com). You should see a secure connection.

---

This guide provides an easy-to-follow process for setting up Nginx for a subdomain and securing it with SSL. Feel free to adapt the configuration to your specific needs and contribute improvements.
