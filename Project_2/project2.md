## LEMP STACK IMPLEMENTATION IN AWS

### This project shows implementation of LEMP on AWS

#### LEMP - Linux, Nginx, Mysql,PHP

Virtual server running on Ubuntu OS is required just like in Project 1.

Update a list of packages in package manager.

``` sudo apt update -y ```

##### Install Nginx

``` sudo apt install nginx ```

Check successful installation of nginx

``` sudo systemctl status nginx ```

OUTPUT:
Project_2\images\Nginx running.jpg


Web server successfully launched in the cloud.
Open the required inbound security to allow traffic to the server - TCP port 80

check access to the server using the shell -

``` curl http://localhost:80 or curl http://127.0.0.1:80 ```

Project_2\images\Welcome to Nginx.jpg

Access through internet may also be done through us of the IP address -

``` http://<Public-IP-Address>:80 ```

public-ip-address is seen in the AWS console or can be gotten by using the ``curl`` command -

``curl -s http://169.254.169.254/latest/meta-data/public-ipv4``

Project_2\images\webpage Nginx.jpg


## ------------------- INSTALLING MYSQL---------
``sudo apt install mysql-server -y``

Log in to MySQL console
`` sudo mysql ``

Set a Password for the user
``ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY 'PassWord.1';``

Exit the MySQL shell with:

``mysql> exit``

Run the MySQL interactive script to create secure login for the database

`` sudo mysql_secure_installation ``

Test if you are able to log in 

`` sudo mysql -p ``

Then exit the console

``mysql> exit``

## -------------- INSTALL PHP -----------------

We have installed Nginx to serve our content and MYSQL to store and manage data. Now we install PHP proccess code code and generate dynamic content for the web server.

Nginx requires an external program to handle PHP processing -  php-fpm, which stands for “PHP fastCGI process manager”

Also required is php-mysql, a PHP module that allows PHP to communicate with MySQL-based databases. 

Core PHP packages will automatically be installed as dependencies.

To install this 2 packages at once, run:

``sudo apt install php-fpm php-mysql``

When prompted, type ``Y`` and press ``ENTER`` to confirm installation.

## ----------Configuring Nginx to use PHP processor--------------

Domain name for this project is ``projectLEMP``

We will need a server block for our domain name, similar to virtual hosts created in the Apache mode. Server block will contain all config dtails and can host more than one domain.

Create a root web directory for your domain (projectLEMP)

``sudo mkdir /var/www/projectLEMP``

Modify ownership of the directory

`` sudo chown -R $USER:$USER /var/www/projectLEMP ``

Then, open  a new config file in the Nginx's `` sites-available`` directory using preffered CLI. Here we use ``nano``

sudo nano /etc/nginx/sites-available/projectLEMP


This creates a blank file. Copy and past the code below into the file

``
#/etc/nginx/sites-available/projectLEMP

server {
    listen 80;
    server_name projectLEMP www.projectLEMP;
    root /var/www/projectLEMP;

    index index.html index.htm index.php;

    location / {
        try_files $uri $uri/ =404;
    }

    location ~ \.php$ {
        include snippets/fastcgi-php.conf;
        fastcgi_pass unix:/var/run/php/php8.1-fpm.sock;
     }

    location ~ /\.ht {
        deny all;
    }

}
``

To **save** and **close** the file, type ``CTRL+X`` and then ``y`` and ``ENTER`` to confirm.


Activate the configuration by linking to the config file from Nginx's ``sites-enabled`` directory:


`` sudo ln -s /etc/nginx/sites-available/projectLEMP /etc/nginx/sites-enabled/ `` 

This will tell Nginx to use the configuration next time it is reloaded

You can test your configuration for syntax errors by typing:

`` sudo nginx -t `` 

You will gwt the following messages if successful

`` nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful `` 

If any errors are reported, go back to your configuration file to review its contents before continuing


we need to disable default Nginx host that is currently configured to listen on port 80, run:

`` sudo unlink /etc/nginx/sites-enabled/default ``

Then reload Nginx to apply the changes:

`` sudo systemctl reload nginx ``

Now the website is now active but the web root /var/www/projectLEMP is still empty. Create an index.html file in that location so we can test the  new server block works as expected:

`` 
sudo echo 'Hello LEMP from hostname' $(curl -s http://169.254.169.254/latest/meta-data/public-hostname) 'with public IP' $(curl -s http://169.254.169.254/latest/meta-data/public-ipv4) > /var/www/projectLEMP/index.html
`` 
Go to your browser and open the website URL using IP address:

`` http://<Public-IP-Address>:80 ``

It's successful if you see a text similar to the one seen from the '**echo**' command in the index.html file.

`` http://<Public-DNS-Name>:80 `` 

You can leave this file in place as a temporary landing page for your application until you set up an ``index.php`` file to replace it. Once you do that, remember to remove or rename the ``index.html`` file from your document root, as it would take precedence over an ``index.php`` file by default.

Your LEMP stack is now fully configured. In the next step, we’ll create a PHP script to test that Nginx is in fact able to handle .php files within your newly configured website.


## ---------------TESTING PHP WITH NGINX ---------------

To confirm that Nginx can handle .php files adequately
### Create a Test PHP file

``sudo nano /var/www/projectLEMO/info.php``

Type or paste the code below in to the file created. This PHP code returns information about the server:


``
<?php
phpinfo();
``

Access the page in the web browser by using the domnain name or public IP address you've set in your Nginx config file, followed by /info.php:

`` http://'server_domain_or_IP'/info.php ``

The output will show a web page that contains details about your server.


To remove the file:

`` sudo rm /var/www/your_domain/info.php ``

You can always regenerate this file if you need it later.



## -------- RETRIEVING DATA FROM MYSQL DATABASE WITH PHP

Here we create a database -a simple To do list. 
We will configure access to this database and let Nginx website query and display data from it.

We will create a database named **john_database** and user **john_user**

But first, we log in to the mysql console:

`` sudo mysql ``

Create the database:

`` mysql> CREATE DATABASE `john_database`; ``


Create a user and grant access:

`` mysql>  CREATE USER 'john_user'@'%' IDENTIFIED WITH mysql_native_password BY 'password'; ``

Grant access:

`` mysql> GRANT ALL ON john_database.* TO 'john_user'@'%'; ``

Exit the shell:

``mysql> exit``

Test if user has proper permission to the MYSQL:

`` mysql -u john_user -p ``

*NB*: Custom database and user was used in the project.

Notice the -p flag in this command, which will prompt you for the password used when creating the example_user user. After logging in to the MySQL console, confirm that you have access to the example_database database:

`` mysql> SHOW DATABASES;``


This will give the following output:

```
Output
+--------------------+
| Database           |
+--------------------+
| john_database   |
| information_schema |
+--------------------+
2 rows in set (0.000 sec)
```

Next, create a table named todo_list.
From the MYSQL console, run the following statement:

```
CREATE TABLE john_database.todo_list (
mysql>     item_id INT AUTO_INCREMENT,
mysql>     content VARCHAR(255),
mysql>     PRIMARY KEY(item_id)
mysql> );
```

INSERT content into the table. Repeat the command a few times using different VALUES:

mysql> INSERT INTO example_database.todo_list (content) VALUES ("My first important item");

Confirm data was successfully added to the table, run:

``mysql>  SELECT * FROM john_database.todo_list;``

You'll see the following output:

```
Output
+---------+--------------------------+
| item_id | content                  |
+---------+--------------------------+
|       1 | My first important item  |
|       2 | My second important item |
|       3 | My third important item  |
|       4 | and this one more thing  |
+---------+--------------------------+
4 rows in set (0.000 sec)
```

After confirming you have a valid data in your test table, exit the MySQL console:

``mysql> exit ``


CREATE a PHP script that will connect to the MYSQL database and query for your content.
Create a new PHP fie in the custom web root directory, use vi editor.

`` nano /var/www/projectLEMP/todo_list.php ``

The following PHP script connects to the MySQL database and queries for the content of the todo_list table, displays the results in a list. If there is a problem with the database connection, it will throw an exception.

Copy the code below to the todo_list.php script:

```
<?php
$user = "john_user";
$password = "password";
$database = "john_database";
$table = "todo_list";

try {
  $db = new PDO("mysql:host=localhost;dbname=$database", $user, $password);
  echo "<h2>TODO</h2><ol>";
  foreach($db->query("SELECT content FROM $table") as $row) {
    echo "<li>" . $row['content'] . "</li>";
  }
  echo "</ol>";
} catch (PDOException $e) {
    print "Error!: " . $e->getMessage() . "<br/>";
    die();
}
```

Save and close after editing

You can access the page in the web browser by visiting the domain name or public IP address confired for the website followed by todo_list.php:

``` http://<Public_domain_or_IP>/todo_list.php ```

You should see a page like this, showing the content you’ve inserted in your test table:




That means your PHP environment is ready to connect and interact with your MySQL server.

In this guide, we have built a flexible foundation for serving PHP websites and applications to your visitors, using Nginx as web server and MySQL as database management system.

