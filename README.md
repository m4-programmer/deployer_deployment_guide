# deployer_deployment_guide
this guide, documents the process i follow to deploy my laravel projects that uses deployer to the server


# Step 1:
First set up ssh access to the server so u can access your server from ur local
1. run the following command on your local machine to generate the SSH key. Note that the -f specifies the filename of the key file, and you can replace gitkey with your own
``` ssh-keygen -t rsa -b 4096 -f  ~/.ssh/gitkey ```
2. It is possible that you have more SSH keys on your local machine, so configure the SSH client to know which SSH private key to use when it connects to your Git server.

Create an SSH config file on your local machine: ``` touch ~/.ssh/config ```
Open the file and add a shortcut to your Git server. This should contain the HostName directive (pointing to your Git server’s hostname) and the IdentityFile directive (pointing to the file path of the SSH key you just created:
```
Host mygitserver.com
    HostName mygitserver.com
    IdentityFile ~/.ssh/gitkey
```
Save and close the file, and then restrict its permissions:
```
chmod 600 ~/.ssh/config
```
Now your SSH client will know which private key use to connect to the Git server.

```
cat ~/.ssh/gitkey.pub
```
Copy the output and add the public key to your Git server.
Now you will be able to connect to your Git server with your local machine. Test the connection with the following command:
```ssh -T git@github.com```
3. On your server as the non root user run the following:
``` nano ~/.ssh/authorized_keys ```
Paste the public key to the editor and hit CTRL-X, Y, then ENTER to save and exit.
- *Note: the public key should be on one line for it to work*

Restrict the permissions of the file:
```chmod 600 ~/.ssh/authorized_keys ```

 4. the step above will work both for adding ssh access to ur github and your server.

# Configuring Deployer User

1. Log in to your LEMP server with a sudo non-root user and create a new user called “deployer” or anything of your choice with the following command: ```sudo adduser deployer ```
2. Laravel needs some writable directories to store cached files and uploads, so the directories created by the deployer user must be writable by the Nginx web server. Add the user to the www-data group to do this: ``` sudo usermod -aG www-data deployer ```
3. The default permission for files created by the deployer user should be 644 for files and 755 for directories. This way, the deployer user will be able to read and write the files, while the group and other users will be able to read them.

Do this by setting deployer’s default umask to 022: ``` sudo chfn -o umask=022 deployer ```
3. We’ll store the application in the /var/www/html/ directory, so change the ownership of the directory to the deployer user and www-data group.
``` sudo chown deployer:www-data /var/www/html ```
4. The deployer user needs to be able to modify files and folders within the /var/www/html directory. Given that, all new files and subdirectories created within the /var/www/html directory should inherit the folder’s group id (www-data). To achieve this, set the group id on this directory with the following command: ``` sudo chmod g+s /var/www/html ```

# Configuring Nginx

1. ``` sudo nano /etc/nginx/sites-available/service.com ``` put the name u want for the nginx config.
2. nginx config block:
```
server {
        listen 80;
        listen [::]:80;

        root /var/www/html/current/public;
        index index.php index.html index.htm;

        server_name example.com www.example.com;

        location / {
                try_files $uri $uri/ /index.php?$query_string;
        }


        location ~ \.php$ {
                include snippets/fastcgi-php.conf;

                fastcgi_param SCRIPT_FILENAME $realpath_root$fastcgi_script_name;
                fastcgi_param DOCUMENT_ROOT $realpath_root;

                fastcgi_pass unix:/run/php/php8.3-fpm.sock;

        }

        location ~ /\.ht {
                deny all;
        }

}
```

3. Save and close the file (CTRL-X, Y, then ENTER), and then enable the new server block by creating a symbolic link to the sites-enabled directory: ``` sudo ln -s /etc/nginx/sites-available/example.com /etc/nginx/sites-enabled/ ```
4. Test your configuration file for syntax errors: ``` sudo nginx -t ```
5. If you see any errors, go back and recheck your file before continuing.

Restart Nginx to push the necessary changes: ``` sudo systemctl restart nginx ```

# Configuring Mysql
1. ``` sudo mysql ```
2. Next, create a new database for the application: ``` CREATE DATABASE my_database DEFAULT CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci; ```
3. Creae user and password: ``` CREATE USER 'user'@'localhost' IDENTIFIED BY 'password'; ```
4. grant privileges: ``` GRANT ALL ON my_database.* TO 'user'@'localhost'; ```
5. reload the privileges: ``` FLUSH PRIVILEGES; ```
6. exit from mySql: ``` EXIT; ```;


# Setting Up the Server with the neccessary apps.

- A: Setting up Firewall
  ``` ufw app list ```
  ``` ufw allow OpenSSH ```
  ``` ufw enable ```
  ``` ufw status ```

  - B: Setting Up Nginx Web Server
    1. ``` sudo apt update```
    2. ``` sudo apt install nginx ```
    3. ``` sudo ufw app list ```
    4. ``` sudo ufw allow 'Nginx Full ```
    5. ``` sudo ufw status ```
   
  - C: Installing MySQl
    1. ``` sudo apt install mysql-server -y ```
    2. ``` sudo mysql_secure_installation ```
    3. consult the settings above for it to set it up
   
  - D: Installing PHP
      1. ```sudo apt install php8.3-fpm php-mysql -y```
      2. Create the root web directory for your_domain as follows (if u are not using the default html folder: ``` sudo mkdir /var/www/your_domain ```
      3. ``` sudo chown -R $USER:$USER /var/www/your_domain ``` Next, assign ownership of the directory with the $USER environment variable, which will reference your current system user
      4. then u can setup the nginx config if u are yet to do so using the reference above.
   
  - E: Setting up Composer
    - In addition to dependencies that should be already included within your Ubuntu 22.04 system, such as git and curl, Composer requires php-cli in order to execute PHP scripts in the command line, and unzip to extract zipped archives. We’ll install these dependencies now.
    1. ``` sudo apt update ```
    2. ``` sudo apt install php-cli unzip ```
    3. ```
       cd ~
        curl -sS https://getcomposer.org/installer -o /tmp/composer-setup.php
       ```
    4. ``` HASH=`curl -sS https://composer.github.io/installer.sig` ```
    5. ``` echo $HASH ```
    6.  ``` php -r "if (hash_file('SHA384', '/tmp/composer-setup.php') === '$HASH') { echo 'Installer verified'; } else { echo 'Installer corrupt'; unlink('composer-setup.php'); } echo PHP_EOL;" ```
    7.  do this to install composer globally: ``` sudo php /tmp/composer-setup.php --install-dir=/usr/local/bin --filename=composer```

  - F: other important installs for smooth runnings
      1. ``` sudo apt-get install php8.3-mbstring php8.3-xml ```
      2. ``` sudo apt install acl  ```
