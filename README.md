# LEMP-WEB-STACK-IMPLEMENTATION
# PROJECT 2 - LEMP WEB STACK IMPLEMENTATION 

## Project Pre-requisite
I already had an account with AWS
I launched an EC2 Instance of t2.micro family with Ubuntu Server 20.04 LTS (HVM) 64bit
Launch GitBash Terminal, cd into the folder that has your project key pair and run following command:

```
ssh -i <key.pem> ubuntu@<Public-IP-address>
```
<img width="450" alt="image" src="https://user-images.githubusercontent.com/88560609/196214486-03850a23-40a6-41b3-aa55-b5189630b791.png">

## STEP 1 – INSTALLING THE NGINX WEB SERVER
 
Installing the Nginx Web Server in order to display web pages to our site visitors. To employ Nginx, a high-performance web server, used the apt package manager to install this package.

Since this is our first time using apt for this session, start off by updating your server package index. Following that, used `apt` install to get Nginx installed:

```
sudo apt update
sudo apt install nginx -y
```

Once the installation is finished, the Nginx web server will be active and running on your Ubuntu 20.04 server.

To verify that nginx was successfully installed and is running as a service in Ubuntu, run:

```
sudo systemctl status nginx
```

If it is green and running, then you did everything correctly – you have just launched your first Web Server in the Clouds!

<img width="451" alt="image" src="https://user-images.githubusercontent.com/88560609/196215362-a450dd5b-d46d-4710-a2e9-c4ac1194477d.png">

Before we can receive any traffic by our Web Server, we need to open TCP port 80 which is default port that web brousers use to access web pages in the Internet.

As we know, we have TCP port 22 open by default on our EC2 machine to access it via SSH, so we need to add a rule to EC2 configuration to open inbound connection through port 80:

<img width="881" alt="image" src="https://user-images.githubusercontent.com/88560609/196216067-e654d607-eb97-4c01-9eee-c56e0de5e4bd.png">

Our server is running and we can access it locally and from the Internet (Source 0.0.0.0/0 means ‘from any IP address’).

First, let us try to check how we can access it locally in our Ubuntu shell, run: and q to quit the interface 

```
curl http://localhost:80
or
curl http://127.0.0.1:80
```

These 2 commands above actually do pretty much the same – they use ‘curl’ command to request our Nginx on port 80 
(actually you can even try to not specify any port – it will work anyway). The difference is that: in the first case 
we try to access our server via DNS name and in the second one – by IP address (in this case IP address 127.0.0.1 corresponds to DNS name ‘localhost’ 
and the process of converting a DNS name to IP address is called "resolution"). 


Now it is time to test how our Nginx server can respond to requests from the Internet.
Open a web browser of your choice and try to access following url

```
http://<Public-IP-Address>:80
```

Another way to retrieve your Public IP address, other than to check it in AWS Web console, is to use following command:

```
curl -s http://169.254.169.254/latest/meta-data/public-ipv4
```

The URL in browser shall also work if you do not specify port number since all web browsers use port 80 by default.

If you see following page, then your web server is now correctly installed and accessible through your firewall.

<img width="928" alt="image" src="https://user-images.githubusercontent.com/88560609/196216698-9db62bbb-11b9-45b3-bdcc-0b8798782a08.png">

## STEP 2 — INSTALLING MYSQL

Now that you have a web server up and running, you need to install a Database Management System (DBMS) to be able to store and manage data for your site in a relational database.
MySQL is a popular relational database management system used within PHP environments, so we will use it in this project.

Again, use apt to acquire and install this software:

```
sudo apt install mysql-server -y
```

When you’re finished, test if you’re able to log in to the MySQL console by typing:


```
sudo mysql
```

<img width="443" alt="image" src="https://user-images.githubusercontent.com/88560609/196217109-927b58b3-3b84-47fc-85d4-c587156638a5.png">

This will connect to the MySQL server as the administrative database user root, which is inferred by the use of sudo when running this command. 
You should see output like this:


To exit the MySQL console, type:

```
mysql> exit
```


Notice that you didn’t need to provide a password to connect as the root user, you can define one by running the `sudo mysql_secure_installation` script. 
That is because the default authentication method for the administrative MySQL user is unix_socket instead of password. Even though this might look like 
a security concern at first, it makes the database server more secure because the only users allowed to log in as the root MySQL user are the system users 
with sudo privileges connecting from the console or through an application running with the same privileges. In practical terms, that means you won’t be 
able to use the administrative database root user to connect from your PHP application. Setting a password for the root MySQL account works as a safeguard, 
in case the default authentication method is changed from unix_socket to password.

For increased security, it’s best to have dedicated user accounts with less expansive privileges set up for every database, especially if you plan on 
having multiple databases hosted on your server.

MySQL server is now installed and secured. Next, we will install PHP, the final component in the LEMP stack.

## STEP 3 – INSTALLING PHP

You have Nginx installed to serve your content and MySQL installed to store and manage your data. 
Now you can install PHP to process code and generate dynamic content for the web server.

While Apache embeds the PHP interpreter in each request, Nginx requires an external program to handle PHP processing and act as a bridge between 
the PHP interpreter itself and the web server. This allows for a better overall performance in most PHP-based websites, but it requires additional configuration. 
You’ll need to install php-fpm, which stands for “PHP fastCGI process manager”, and tell Nginx to pass PHP requests to this software for processing. 
Additionally, you’ll need php-mysql, a PHP module that allows PHP to communicate with MySQL-based databases. Core PHP packages will automatically be installed as dependencies.

To install these 2 packages at once, run:

```
sudo apt install php-fpm php-mysql -y
```

You now have your PHP components installed. Next, you will configure Nginx to use them.

## STEP 4 — CONFIGURING NGINX TO USE PHP PROCESSOR

When using the Nginx web server, we can create server blocks (similar to virtual hosts in Apache) to encapsulate configuration details and host more than one domain on a single server. In this project, we will use projectLEMP as an example domain name.

On Ubuntu 20.04, Nginx has one server block enabled by default and is configured to serve documents out of a directory at /var/www/html. 
While this works well for a single site, it can become difficult to manage if you are hosting multiple sites. Instead of modifying /var/www/html, we’ll create a directory structure within /var/www for the your_domain website, leaving /var/www/html in place as the default directory to be served if a client request does not match any other sites.

Create the root web directory for your_domain as follows:

```
sudo mkdir /var/www/projectLEMP
```

Next, assign ownership of the directory with the $USER environment variable, which will reference your current system user:

```
sudo chown -R $USER:$USER /var/www/projectLEMP
```

Then, open a new configuration file in Nginx’s sites-available directory using your preferred command-line editor. Here, we’ll use nano:

```
sudo nano /etc/nginx/sites-available/projectLEMP
```

This will create a new blank file. Paste in the following bare-bones configuration:


```
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
        fastcgi_pass unix:/var/run/php/php7.4-fpm.sock;
     }

    location ~ /\.ht {
        deny all;
    }

}

```

<img width="518" alt="image" src="https://user-images.githubusercontent.com/88560609/196218563-6e5c5358-d05e-4b7c-80e9-17bec9788fdf.png">

When you’re done editing, save and close the file. If you’re using nano, you can do so by typing CTRL+X and then y and ENTER to confirm.

Here’s what each of these directives and location blocks do:


1. listen — Defines what port Nginx will listen on. In this case, it will listen on port 80, the default port for HTTP.

2. root — Defines the document root where the files served by this website are stored.

3. index — Defines in which order Nginx will prioritize index files for this website. 
It is a common practice to list index.html files with a higher precedence than index.
php files to allow for quickly setting up a maintenance landing page in PHP applications. You can adjust these settings to better suit your application needs.

4. server_name — Defines which domain names and/or IP addresses this server block should respond for. Point this directive to your server’s domain name or public IP address.

5. location / — The first location block includes a try_files directive, which checks for the existence of files or directories matching a URI request. If Nginx cannot find the appropriate resource, it will return a 404 error.

6. location ~ \.php$ — This location block handles the actual PHP processing by pointing Nginx to the fastcgi-php.conf configuration file and the php7.4-fpm.sock file, which declares what socket is associated with php-fpm.

7. location ~ /\.ht — The last location block deals with .htaccess files, which Nginx does not process.
By adding the deny all directive, if any .htaccess files happen to find their way into the document root ,they will not be served to visitors.
When you’re done editing, save and close the file. If you’re using nano, you can do so by typing CTRL+X and then y and ENTER to confirm.

Activate your configuration by linking to the config file from Nginx’s sites-enabled directory:

```
sudo ln -s /etc/nginx/sites-available/projectLEMP /etc/nginx/sites-enabled/
```

This will tell Nginx to use the configuration next time it is reloaded. You can test your configuration for syntax errors by typing:

```
sudo nginx -t
```

You shall see following message:

(nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful)

<img width="521" alt="image" src="https://user-images.githubusercontent.com/88560609/196219189-a55dcc30-75d6-4db5-9082-d47b6d1aa5e3.png">

If any errors are reported, go back to your configuration file to review its contents before continuing.

We also need to disable default Nginx host that is currently configured to listen on port 80, for this run:

```
sudo unlink /etc/nginx/sites-enabled/default
```

When you are ready, reload Nginx to apply the changes:

```
sudo systemctl reload nginx
```

Your new website is now active, but the web root /var/www/projectLEMP is still empty.
Create an index.html file in that location so that we can test that your new server block works as expected:

```
sudo echo 'Hello LEMP from hostname' $(curl -s http://169.254.169.254/latest/meta-data/public-hostname) 'with public IP' $(curl -s http://169.254.169.254/latest/meta-data/public-ipv4) > /var/www/projectLEMP/index.html
```

Now go to your browser and try to open your website URL using IP address:


```
http://<Public-IP-Address>:80
```


If you see the text from ‘echo’ command you wrote to index.html file, then it means your Nginx site is working as expected.
In the output you will see your server’s public hostname (DNS name) and public IP address. You can also access your website in your browser by public DNS name,
not only by IP – try it out, the result must be the same (port is optional)

```
http://<Public-DNS-Name>:80
```

<img width="647" alt="image" src="https://user-images.githubusercontent.com/88560609/196221753-72b9faf3-9c72-451f-a93c-519de03be67a.png">

You can leave this file in place as a temporary landing page for your application until you set up an index.php file to replace it. 
Once you do that, remember to remove or rename the index.html file from your document root, as it would take precedence over an index.php file by default.

Your LEMP stack is now fully configured. In the next step, we’ll create a PHP script to test that Nginx is in fact able to handle .php files within your newly configured website.


## STEP 5 – TESTING PHP WITH NGINX


Your LEMP stack should now be completely set up.
At this point, LAMP stack is completely installed and fully operational.
Test it to validate that Nginx can correctly hand .php files off to your PHP processor.
Creating a test PHP file in document root. Open a new file called info.php within your document root in your text editor:

```
sudo nano /var/www/projectLEMP/info.php
```

Type or paste the following lines into the new file. This is valid PHP code that will return information about your server:

```
<?php
phpinfo();
```

<img width="523" alt="image" src="https://user-images.githubusercontent.com/88560609/196220806-f44525f8-268e-4a90-9d3e-44309b60c375.png">


You can now access this page in a web browser by visiting the domain name or public IP address in the Nginx configuration file, followed by /info.php:


```
http://<server_domain_or_IP>/info.php
```

You will see a web page containing detailed information about your server:

![image](https://user-images.githubusercontent.com/88560609/200819576-2b65e80c-9270-43e5-a393-cc92c0d21dd9.png)


After checking the relevant information about your PHP server through that page, it’s best to remove the file you created as it contains
sensitive information about your PHP environment and your Ubuntu server. You can use `rm` to remove that file:


```
sudo rm /var/www/projectLEMP/info.php
```

You can always regenerate this file if you need it later.


## STEP 6 – RETRIEVING DATA FROM MYSQL DATABASE WITH PHP (CONTINUED)


In this step, create a test database (DB) with simple "To do list" and configure access to it, so the Nginx website would be able to query data from the DB and display it.

At the time of this writing, the native MySQL PHP library mysqlnd doesn’t support caching_sha2_authentication, the default authentication method for MySQL 8.
We’ll need to create a new user with the mysql_native_password authentication method in order to be able to connect to the MySQL database from PHP.

We will create a database named example_database and a user named example_user, but you can replace these names with different values.

First, connect to the MySQL console using the root account:


```
sudo mysql
```

To create a new database, run the following command from your MySQL console:


```
CREATE DATABASE example_database;
```

Now you can create a new user and grant him full privileges on the database you have just created.

The following command creates a new user named **example_user**, using mysql_native_password as default authentication method.
We’re defining this user’s password as **password**, but you should replace this value with a secure password of your own choosing.

```
CREATE USER 'example_user'@'%' IDENTIFIED WITH mysql_native_password BY 'password';
```

Now we need to give this user permission over the example_database database:

```
GRANT ALL ON example_database.* TO 'example_user'@'%';
```

This will give the example_user user full privileges over the example_database database, while preventing this user from creating or modifying other databases on your server.

Now exit the MySQL shell with:

```
exit
```

<img width="528" alt="image" src="https://user-images.githubusercontent.com/88560609/196227868-dc65f402-3e1e-4424-b33c-28f16d8e3d47.png">

You can test if the new user has the proper permissions by logging in to the MySQL console again, this time using the custom user credentials:

```
mysql -u example_user -p
```

Notice the -p flag in this command, which will prompt you for the password used when creating the example_user user, which is `password`
After logging in to the MySQL console, confirm that you have access to the example_database database:


```
SHOW DATABASES;
```

This will give you the following output:

```
Output
+--------------------+
| Database           |
+--------------------+
| example_database   |
| information_schema |
+--------------------+
2 rows in set (0.000 sec)
```

<img width="520" alt="image" src="https://user-images.githubusercontent.com/88560609/196228496-531752e6-ff04-459b-adf3-7b88a60415c5.png">

Next, we’ll create a test table named todo_list. From the MySQL console, run the following statement:

```
CREATE TABLE example_database.todo_list (item_id INT AUTO_INCREMENT, content VARCHAR(255), PRIMARY KEY(item_id));
```

Insert a few rows of content in the test table. You might want to repeat the next command a few times, using different VALUES:

```
INSERT INTO example_database.todo_list (content) VALUES ("My first important item");

```

To confirm that the data was successfully saved to your table, run:

```
SELECT * FROM example_database.todo_list;
```

You’ll see the following output:


```
Output
+---------+-------------------------+
| item_id | content                 |
+---------+-------------------------+
|       1 | My first important item |
+---------+-------------------------+
1 row in set (0.00 sec)

```

<img width="538" alt="image" src="https://user-images.githubusercontent.com/88560609/196229910-cbffc47e-5b6e-4c4b-90b2-4a6771eb62dc.png">

After confirming that you have valid data in your test table, you can exit the MySQL console:

```
exit
```

Now you can create a PHP script that will connect to MySQL and query for your content. Create a new PHP file in your custom web root directory using your preferred editor. 
We’ll use vi for that:


```
nano /var/www/projectLEMP/todo_list.php
```

The following PHP script connects to the MySQL database and queries for the content of the todo_list table, displays the results in a list. 
If there is a problem with the database connection, it will throw an exception.

Copy this content into your todo_list.php script:


```
<?php
$user = "example_user";
$password = "password";
$database = "example_database";
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


Save and close the file when you are done editing.

You can now access this page in your web browser by visiting the domain name or public IP address configured for your website, followed by /todo_list.php:


```
http://<Public_domain_or_IP>/todo_list.php
```

You should see a page like this, showing the content you’ve inserted in your test table:

![image](https://user-images.githubusercontent.com/88560609/200821203-46432a65-6019-4b3e-ac6f-653b43faaaf2.png)


That means your PHP environment is ready to connect and interact with your MySQL server.

THE END
