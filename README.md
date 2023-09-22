# Initial Server Setup with Ubuntu 20.04

## Introduction
When you first create a new Ubuntu 20.04 server, you should perform some important configuration steps as part of the basic setup. These steps will increase the security and usability of your server, and will give you a solid foundation for subsequent actions.
[https://www.digitalocean.com/community/tutorials/initial-server-setup-with-ubuntu-20-04](https://www.digitalocean.com/community/tutorials/initial-server-setup-with-ubuntu-20-04)

## Step 1 — Logging in as root
If you are not already connected to your server, log in now as the **root** user using the following command (substitute the highlighted portion of the command with your server’s public IP address):
```bash
ssh root@your_server_ip
```

## Step 2 — Creating a new user
This example creates a new user called tocaan:
```bash
adduser tocaan
```

## Step 3 — Granting Administrative Privileges
As **root**, run these commands to add your new user to the **sudo** and **www-data** groups:
```bash
usermod -aG sudo tocaan

# Also add the newly created user to www-data group
usermod -aG www-data tocaan
```

## Step 4 — Generating a New SSH Key for the User Tocaan
First, change the current user to **tocaan**:
```bash
su - tocaan
```
As **tocaan**, run this command to generate a new SSH key on the server:
```bash
ssh-keygen -t rsa -b 4096
```
Add the content from the public key file i.e. `.pub` to your github repository, `settings > deploy keys > Add deploy keys`:
```bash
cat ~/.ssh/id_rsa.pub
```
Switch back to the root user to complete the installation:
```bash
exit
```

## Step 5 — Installing NGINX Web Server
```bash
# Add PPA for NGINX Stable with HTTP/2
sudo add-apt-repository ppa:ondrej/nginx
sudo apt update

sudo apt install nginx
```

## Step 6 — Setting Up a Basic Firewall
As **root**, run these commands to enable the firewall and allow **NGINX** and **OpenSSH** ports only:
```bash
# List available apps
ufw app list

# Allow 22, 80 and 443 ports only
ufw allow OpenSSH
ufw allow 'Nginx Full'

# Start the firewall
ufw enable

# Check the status
ufw status
```

## Step 7 — Installing MySQL Server
```bash
sudo apt install mysql-server-8.0
```
When the installation is finished, it’s recommended that you run a security script that comes pre-installed with MySQL. This script will remove some insecure default settings and lock down access to your database system. Start the interactive script by running:
```bash
# Improve the security of MySQL installation
sudo mysql_secure_installation
```

### (Important) Adjusting User Authentication and Privileges
In Ubuntu systems running MySQL 5.7 (and later versions), the  **root**  MySQL user is set to authenticate using the  `auth_socket`  plugin by default rather than with a password. This allows for some greater security and usability in many cases, but it can also complicate things when you need to allow an external program (e.g., phpMyAdmin) to access the user.

In order to use a password to connect to MySQL as  **root**, you will need to switch its authentication method from  `auth_socket`  to  `mysql_native_password`. To do this, open up the MySQL prompt from your terminal:
```bash
sudo mysql
```

To configure the **root** account to authenticate with a password, run the following `ALTER USER` command. Be sure to change `password` to a strong password of your choosing:
```bash
mysql> ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY 'password';
```
Then, run `FLUSH PRIVILEGES` which tells the server to reload the grant tables and put your new changes into effect:
```bash
mysql> FLUSH PRIVILEGES;
```

## Step 8 — Installing Required PHP 7.4 Modules
You have **Nginx** installed to serve your content and **MySQL** installed to store and manage your data. Now you can install **PHP** to process code and generate dynamic content for the web server.
```bash
sudo apt install php7.4-bcmath php7.4-bz2 php7.4-cli php7.4-curl php7.4-fpm php7.4-gd php7.4-gmp php7.4-intl php7.4-json php7.4-mbstring php7.4-mysql php7.4-odbc php7.4-opcache php7.4-xml php7.4-zip php7.4-xsl
```

### Installing ImageMagick extension
```bash
sudo apt install php-imagick
```

### Modifying the PHP Configuration
```bash
sudo nano /etc/php/7.4/fpm/conf.d/30-tocaan.ini
```
Paste in the following configuration:
```ini
; Maximum allowed size for uploaded files.
upload_max_filesize = 100M

; Maximum size of POST data that PHP will accept.
post_max_size = 100M

; Maximum execution time of each script, in seconds
max_execution_time = 60

; Maximum amount of memory a script may consume (256MB)
memory_limit = 256M
```
Restart PHP-FPM service to enable the changes:
```bash
sudo service php7.4-fpm restart
```

## Step 9 — Installing phpMyAdmin
Run the following commands to install **phpMyAdmin** to easily manage **MySQL** databases: 
```bash
cd /var/www/html

# Get the latest version URL from here:
# https://www.phpmyadmin.net/downloads/

# Download phpMyAdmin
sudo wget https://files.phpmyadmin.net/phpMyAdmin/5.0.2/phpMyAdmin-5.0.2-all-languages.zip

# Extract the downloaded file to a new directory (./pma)
sudo unzip phpMyAdmin-5.0.2-all-languages.zip
sudo mv phpMyAdmin-5.0.2-all-languages pma

# Delete the downloaded file
sudo rm phpMyAdmin-5.0.2-all-languages.zip

# Change files owner and permissions
cd pma
chown www-data:www-data ./ -R
find . -type d -exec chmod 755 {} \;
find . -type f -exec chmod 644 {} \;
```

## Step 10 — Configuring NGINX to Use the PHP Processor
On Ubuntu 20.04, Nginx has one server block enabled by default and is configured to serve documents out of a directory at `/var/www/html`. 

Open the default file in Nginx’s sites-available directory using your preferred command-line editor. Here, we’ll use nano:
```bash
sudo nano /etc/nginx/sites-available/default
```
Paste in the following bare-bones configuration:
```bash
server {
    listen 80 default_server;
    listen [::]:80 ipv6only=on default_server;

    root /var/www/html;

    server_name _;

    add_header X-Frame-Options "SAMEORIGIN";
    add_header X-XSS-Protection "1; mode=block";
    add_header X-Content-Type-Options "nosniff";

    # Add index.php to the list if you are using PHP
    index index.php index.html index.htm;

    charset utf-8;

    location = /favicon.ico { access_log off; log_not_found off; }
    location = /robots.txt  { access_log off; log_not_found off; }

    location / {
        # First attempt to serve request as file, then
        # as directory, then fall back to displaying a 404.
        # try_files $uri $uri/ =404;
        try_files $uri $uri/ =404;
    }

    # pass PHP scripts to FastCGI server
    #
    location ~ \.php$ {
        include snippets/fastcgi-php.conf;

        # With php-fpm (or other unix sockets):
        fastcgi_pass unix:/run/php/php7.4-fpm.sock;
    #   # With php-cgi (or other tcp sockets):
    #   fastcgi_pass 127.0.0.1:9000;
    }

    # deny access to .htaccess files, if Apache's document root
    # concurs with nginx's one
    #
    location ~ /\.ht {
        deny all;
    }
}
```

You can test your configuration for syntax errors by typing:
```bash
sudo service nginx configtest
```

If any errors are reported, go back to your configuration file to review its contents before continuing.

When you are ready, reload Nginx to apply the changes:
```bash
sudo service nginx reload
```

## Where To Go From Here?
At this point, you have a solid foundation for your server. You can install any of the software you need on your server now.

- [Install Laravel](https://github.com/tocaantech/server-setup/blob/master/Install-Laravel.md)
- [Install WordPress (soon)](https://github.com/tocaantech/server-setup/blob/master/Install-WordPress.md)
# server-setup
# server-setup
# server-setup
