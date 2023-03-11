# Documentation for AWS LAMP STACK implementation

This Projec shows how to implement LAMP (Linux, Apache, MYSQL, PHP) technology on AWS.

Requirements for this project include an AWS account and a virtual server running on Ubuntu OS

Before launching the Ubuntu, install some pre-requisites **OpenSSH**, [Install OpenSSH](https://learn.microsoft.com/en-us/windows-server/administration/openssh/openssh_install_firstuse?tabs=powershell#tabpanel_1_powershell) on your local machine that authenticates connection from your local device to the virtual server.
Launch the Ubuntu virtual server on AWS and update it

to Update, run **sudo apt update**

After update complete, continue with downloading the stacks required as follows -

Install Apache using Ubuntu’s package manager
**sudo apt install apache2**

Verify correct installation of apache2
**sudo systemctl status apache2**

![output of apache2 status]
Project_1\images\209189490-2a3ce645-a71f-4094-8d01-7523a699f8c5.png

**Verify Apache2 is accessible from the server**
The default port for webservers to recieve traffic is Port 80. So edit the inbound rules on the EC2 security config, open TCP Port 80 to enable access to web server.

Server can be accessed locally or through the internet

Access through the internet will be achieved through the use of the IP address

Local connection is achieved through the ```curl``` command

```curl http://localhost:80 or curl http://127.0.0.1:80```

## ------------------------------------- INSTALLING MYSQL----------------------------------------

Install mysql on the ubuntu server

```sudo apt install mysql-server -y```

Log into the MYSQL console

``` sudo mysql ```

OUTPUT:
Project_1\images\mysql log in.png

Configure the database user and log in password for mysql user
 ``` ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY 'PassWord.1'; ```

Exit the shell with ``` exit ```

Run MYSQL interactive script. This script removes some inseure default settings and creates secure accessibilty settings.
To run the script -

``` sudo mysql_secure_installation ```

To ensure access to MYSQL, run -

```sudo mysql -p```

TO exit the console, type ``` exit ```

## ------------------- INSTALLING PHP -------------------

Here we will install 3 components -
PHP - to process code to display dynamic content to the user.
php-mysql -  a PHP module that allows PHP to communicate with MYSQL-based DBs.
libapache2-mod-php - a module that enables apache to handle PHP files.

We can install 3 modules at once, using the command
``` sudo apt install php libapache2-mod-php php-mysql ```

Confirm PHP version
``` php -v ```

Project_1\images\php version.png

At this point, we have successfully installed all applications that make up the LAMP stack

- [x] Linux (Ubuntu)
- [x] Apache HTTP Server
- [x] Mysql
- [x] PHP

## -------------- CREATING  A VIRTUAL HOST FOR YOUR WEBSITE USING APACHE --

We will set up a virtual host for the PHP script. Virtual hosts allows hosting of multiple domains on a single server.
We will set up a domain named ``` staxxlamp ```.

Create a directory for staxxlamp

``` sudo mkdir /var/www/staxxlamp ```

Then, assign ownership of the directory with your current system user:

```sudo chown -R $USER:$USER /var/www/staxxlamp```

Then, create and open a new configuration file in Apache’s sites-available directory

```sudo vi /etc/apache2/sites-available/staxxlamp.conf```

This create a blank file, then paste the code below and save.

```
<VirtualHost *:80>
    ServerName staxxlamp
    ServerAlias www.staxxlamp 
    ServerAdmin webmaster@localhost
    DocumentRoot /var/www/staxxlamp
    ErrorLog ${APACHE_LOG_DIR}/error.log
    CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
```

list the new files in the ``sites-available`` directory`

The output from above should look similar to this:

``000-default.conf  default-ssl.conf  staxxlamp.conf``

So we have set ``/var/www/staxxlamp`` as the root folder of ``staxxlamp``

You can now use ``a2ensite`` command to enable the new virtual host:

``sudo a2ensite staxxlamp``

You might want to disable the default website that comes installed with Apache. This is required if you’re not using a custom domain name, because in this case Apache’s default configuration would overwrite your virtual host. To disable Apache’s default website use ``a2dissite`` command , type:

`` sudo a2dissite 000-default ``

OUTPUT:
Project_1\images\staxxlamp sudo.png

To make sure your configuration file  is clean and doesn’t contain syntax errors, run:

``sudo apache2ctl configtest``

Finally, reload Apache so these changes take effect:

``sudo systemctl reload apache2``

Your new website is now active, but the web root /var/www/staxxlamp is still empty. Create an index.html file in that location so that we can test that the virtual host works as expected:

``sudo echo 'Hello LAMP from hostname' $(curl -s http://169.254.169.254/latest/meta-data/public-hostname) 'with public IP' $(curl -s http://169.254.169.254/latest/meta-data/public-ipv4) > /var/www/staxxlamp/index.html``

OUTPUT:
Project_1\images\sudo apachectl.png

Now go to your browser and try to open your website URL using IP address:

``http://<Public-IP-Address>:80``

If you see the text from ‘echo’ command you wrote to ``index.html`` file, then it means your Apache virtual host is working as expected.

You can also access your website in your browser by public DNS name, not only by IP – try it out, the result must be the same (port is optional)

``http://<Public-DNS-Name>:80``

You can leave this file in place as a temporary landing page for your application until you set up an ``index.php`` file to replace it. Once you do that, remember to remove or rename the ``index.html`` file from your document root, as it would take precedence over an ``index.php`` file by default.

## ------------------------ENABLE PHP ON THE WEBSITE -----------------------------

With the default DirectoryIndex settings on Apache, an ``index.html`` will always take precedence over an ``index.php`` file. This is useful for setting up maintenance pages in PHP applications, by creating a temporary index.html file containing an informative message to visitors.

Lets modify the setting and change precedence to the ``index.php`` file

So, we will edit the ``/etc/apache2/mods-enabled/dir.conf`` file and change the order in which the ``index.php`` file is listed within the ``DirectoryIndex`` directive:

``sudo vim /etc/apache2/mods-enabled/dir.conf``

``
<IfModule mod_dir.c>
        #Change this:
        #DirectoryIndex index.html index.cgi index.pl index.php index.xhtml index.htm
        #To this:
        DirectoryIndex index.php index.html index.cgi index.pl index.xhtml index.htm
</IfModule>
``

Then reload apache to effect changes

``sudo systemctl reload apache2``

### Finally, we will create a PHP script to test that PHP is correctly installed and configured on your server

Now that you have a custom location to host your website’s files and folders, we’ll create a PHP test script to confirm that Apache is able to handle and process requests for PHP files.

Create a PHP file named ``index.php`` inside the ``staxxlamp`` folder (custom web root folder)

``vim /var/www/staxxlamp/index.php``

This will open a blank file. Add the following text, which is valid PHP code, inside the file:

```
<?php
phpinfo();
```

save and close the file. Refresh the webpage, the result should be SIMILAR to the image below;

Project_1\images\phpinfo.png

It is advisable to remove the file as it contains sensitive information about your server and php site config.

**`sudo rm /var/www/staxxlamp/index.php`**

**Thank you!!**
