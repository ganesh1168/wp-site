# Automated Deployment for WordPress with Nginx, LEMP Stack, and GitHub Actions

This README provides a step-by-step guide to set up an automated deployment process for a WordPress website using Nginx as the web server, the LEMP (Linux, Nginx, MySQL, PHP) stack, and GitHub Actions as the CI/CD automation tool. The deployment process is designed to follow security best practices and ensure optimal performance for your website.

## Prerequisites

Before you begin, ensure you have the following prerequisites in place:

1. A Linux-based server with root access (Ubuntu 20.04 LTS recommended).
2. A domain name pointed to your server's IP address.
3. A GitHub repository containing your WordPress project.
4. Basic knowledge of Linux commands and server administration.

## Step 1: Setting up the Server

1. Connect to your server via SSH:

   ```shell
   ssh username@your_server_ip
   ```

2. Update the system packages:

   ```shell
   sudo apt update
   sudo apt upgrade
   ```

3. Install the LEMP stack (Linux, Nginx, MySQL, PHP) on your server:

   ```shell
   sudo apt install nginx mysql-server php-fpm php-mysql
   ```

4. Secure your MySQL installation:

   ```shell
   sudo mysql_secure_installation
   ```

5. Create a MySQL database and user for your WordPress site:

   ```shell
   mysql -u root -p
   CREATE DATABASE wordpress;
   CREATE USER 'wordpressuser'@'localhost' IDENTIFIED BY 'your_password';
   GRANT ALL PRIVILEGES ON wordpress.* TO 'wordpressuser'@'localhost';
   FLUSH PRIVILEGES;
   EXIT;
   ```

6. Configure Nginx to serve your WordPress site. Create a new server block configuration file:

   ```shell
   sudo nano /etc/nginx/sites-available/wordpress
   ```

   Add the following configuration (replace `your_domain.com` with your domain name):

   ```nginx
   server {
       listen 80;
       server_name your_domain.com www.your_domain.com;

       root /var/www/html/wordpress;
       index index.php;

       location / {
           try_files $uri $uri/ /index.php?$args;
       }

       location ~ \.php$ {
           include snippets/fastcgi-php.conf;
           fastcgi_pass unix:/var/run/php/php7.4-fpm.sock; # Use the correct PHP version
       }

       location ~ /\.ht {
           deny all;
       }
   }
   ```

7. Enable the Nginx server block:

   ```shell
   sudo ln -s /etc/nginx/sites-available/wordpress /etc/nginx/sites-enabled/
   ```

8. Test the Nginx configuration:

   ```shell
   sudo nginx -t
   ```

9. Reload Nginx to apply the changes:

   ```shell
   sudo systemctl reload nginx
   ```

10. Download WordPress and configure it:

    ```shell
    cd /var/www/html
    sudo wget https://wordpress.org/latest.tar.gz
    sudo tar -xzvf latest.tar.gz
    sudo mv wordpress/* .
    sudo chown -R www-data:www-data /var/www/html/
    sudo cp wp-config-sample.php wp-config.php
    sudo nano wp-config.php
    ```

    Update the database configuration with the database name, username, and password you created earlier.

11. Secure your WordPress installation:

    ```shell
    sudo mkdir /var/www/html/wp-content/uploads
    sudo chmod 775 /var/www/html/wp-content/uploads
    sudo apt install certbot python3-certbot-nginx
    sudo certbot --nginx -d your_domain.com -d www.your_domain.com
    ```

12. Test your WordPress site by accessing it via your domain name in a web browser.

## Step 2: Setting up GitHub Actions for CI/CD

1. Create a `.github/workflows/main.yml` file in your GitHub repository with the following content (adjust as needed):

```yaml
name: Deploy to Remote Server
on:
  push:
    branches:
      - main # Change this to the branch you want to trigger deployments from

jobs:
  deploy:
    runs-on: ubuntu-latest # You can choose a different runner if needed

    steps:
    - name: Checkout code
      uses: actions/checkout@v2

    - name: Install SSH key
      run: |
        mkdir -p ~/.ssh
        echo "$SSH_PRIVATE_KEY" > ~/.ssh/id_rsa
        chmod 600 ~/.ssh/id_rsa
        ssh-keyscan -t rsa ${{ secrets.SSH_HOST }} >> ~/.ssh/known_hosts
      env:
        SSH_PRIVATE_KEY: ${{ secrets.SSH_PRIVATE_KEY }} # Store your SSH private key as a GitHub secret

    - name: SSH into the remote server and deploy
      run: |
        ssh -i ~/.ssh/id_rsa ${{ secrets.SSH_USER }}@ ${{ secrets.SSH_HOST }} "cd ${{ secrets.DEPLOY_PATH }} &&  sudo git pull origin main"
 # Modify this command to suit your deployment needs
      env:
        SSH_PRIVATE_KEY: ${{ secrets.SSH_PRIVATE_KEY }}
```

2. Generate an SSH key pair on your local machine if you haven't already:

   ```shell
   ssh-keygen -t rsa -b 4096 -C "your_email@example.com"
   ```

3. Add the SSH public key to your server's `~/.ssh/authorized_keys` file.

4. Add the private key to your GitHub repository's Secrets as `SSH_PRIVATE_KEY`.

## Step 3: Automated Deployment

Now, whenever you push changes to the `main` branch of your GitHub repository, GitHub Actions will automatically deploy your WordPress site to your server.

