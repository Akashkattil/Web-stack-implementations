# Web-stack-implementations
A technology stack is a set of frameworks and tools used to develop a software product. This set of frameworks and tools are very specifically chosen to work together in creating a well-functioning software. They are acronymns for individual technologies used together for a specific technology product.

In order to complete this project you will need an AWS account and a virtual server with Ubuntu Server OS.

## Step 1 — Installing APACHE and updating the firewall
What exactly is Apache?

Apache HTTP Server is the most widely used web server software. Developed and maintained by Apache Software Foundation, Apache is an open source software available for free. It runs on 67% of all webservers in the world. It is fast, reliable, and secure. It can be highly customized to meet the needs of many different environments by using extensions and modules.

Install Apache using Ubuntu’s package manager ‘apt’:
```
#update a list of packages in package manager
sudo apt update

#run apache2 package installation
sudo apt install apache2
```
### To verify that apache2 is running as a Service in our OS, use following command
```
sudo systemctl status apache2
```
If it is green and running, then you did everything correctly – you have just launched your first Web Server in the Clouds!
Before we can receive any traffic by our Web Server, we need to open TCP port 80 which is the default port that web browsers use to access web pages on the Internet
```
curl http://localhost:80
or
curl http://127.0.0.1:80
 ```
Now it is time for us to test how our Apache HTTP server can respond to requests from the Internet. Open a web browser of your choice and try to access following url
```
http://<Public-IP-Address>:80
```
If you see *Apache Ubuntu Default Page*, then your web server is now correctly installed and accessible through your firewall.

## Step 2 — Installing MYSQL
Now that you have a web server up and running, you need to install a Database Management System (DBMS) to be able to store and manage data for your site in a relational database. MySQL is a popular relational database management system used within PHP environments, so we will use it in our project.

Again, use ‘apt’ to acquire and install this software:
```
sudo apt install mysql-server
```
When prompted, confirm installation by typing Y, and then ENTER.

When the installation is finished, log in to the MySQL console by typing:
```
sudo mysql
```
This will connect to the MySQL server as the administrative database user root
Set a password for the root user, using mysql_native_password as default authentication method. We’re defining this user’s password as PassWord.1.
```
ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY 'PassWord.1';
```
Exit the MySQL shell with:
```
mysql> exit
```
Start the interactive script by running:
```
sudo mysql_secure_installation
```
This will ask if you want to configure the VALIDATE PASSWORD PLUGIN.
Note: Enabling this feature is something of a judgment call. If enabled, passwords which don’t match the specified criteria will be rejected by MySQL with an error. It is safe to leave validation disabled, but you should always use strong, unique passwords for database credentials.
Answer Y for yes, or anything else to continue without enabling.
When you’re finished seting password, test if you’re able to log in to the MySQL console by typing:
```
sudo mysql -p
```
Notice the -p flag in this command, which will prompt you for the password used after changing the root user password.
To exit the MySQL console, type:
```
mysql> exit
```
## Step 3 — Installing php 
You have Apache installed to serve your content and MySQL installed to store and manage your data. PHP is the component of our setup that will process code to display dynamic content to the end user. In addition to the php package, you’ll need php-mysql, a PHP module that allows PHP to communicate with MySQL-based databases. You’ll also need libapache2-mod-php to enable Apache to handle PHP files. Core PHP packages will automatically be installed as dependencies.

To install these 3 packages at once, run:
```
sudo apt install php libapache2-mod-php php-mysql
```
Once the installation is finished, you can run the following command to confirm your PHP version:
```
php -v
```

## Step 4 — Creating a virtual host using Apache
In this project, you will set up a domain called *projectlamp*
Create the directory for projectlamp using ‘mkdir’ command as follows:
```
sudo mkdir /var/www/projectlamp
```
Next, assign ownership of the directory with your current system user:
```
sudo chown -R $USER:$USER /var/www/projectlamp
```
Then, create a new configuration file in Apache’s sites-available directory
```
sudo vi /etc/apache2/sites-available/projectlamp.conf
```
This will create a new blank file. Paste in the following bare-bones configuration by hitting on i on the keyboard to enter the insert mode, and paste the text:
```
<VirtualHost *:80>
    ServerName projectlamp
    ServerAlias www.projectlamp 
    ServerAdmin webmaster@localhost
    DocumentRoot /var/www/projectlamp
    ErrorLog ${APACHE_LOG_DIR}/error.log
    CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
```
You can use the ls command to show the new file in the sites-available directory
```
sudo ls /etc/apache2/sites-available
```
You will see something like this;
```
000-default.conf  default-ssl.conf  projectlamp.conf
```
With this VirtualHost configuration, we’re telling Apache to serve projectlamp using /var/www/projectlampl as its web root directory. If you would like to test Apache without a domain name, you can remove or comment out the options ServerName and ServerAlias by adding a # character in the beginning of each option’s lines. Adding the # character there will tell the program to skip processing the instructions on those lines.

You can now use a2ensite command to enable the new virtual host:
```
sudo a2ensite projectlamp
```
You might want to disable the default website that comes installed with Apache. This is required if you’re not using a custom domain name, because in this case Apache’s default configuration would overwrite your virtual host. To disable Apache’s default website use a2dissite command , type:
```
sudo a2dissite 000-default
```
To make sure your configuration file doesn’t contain syntax errors, run:
```
sudo apache2ctl configtest
```
Finally, reload Apache so these changes take effect:
```
sudo systemctl reload apache2
```
Your new website is now active, but the web root /var/www/projectlamp is still empty. Create an index.html file in that location so that we can test that the virtual host works as expected:
```
sudo echo 'Hello LAMP from hostname' $(curl -s http://169.254.169.254/latest/meta-data/public-hostname) 'with public IP' $(curl -s http://169.254.169.254/latest/meta-data/public-ipv4) > /var/www/projectlamp/index.html
```
Now go to your browser and try to open your website URL using IP address:
```
http://<Public-IP-Address>:80
```
## Step 5 — Enable php on the website
With the default DirectoryIndex settings on Apache, a file named index.html will always take precedence over an index.php file. This is useful for setting up maintenance pages in PHP applications, by creating a temporary index.html file containing an informative message to visitors. Because this page will take precedence over the index.php page, it will then become the landing page for the application. Once maintenance is over, the index.html is renamed or removed from the document root, bringing back the regular application page.

In case you want to change this behavior, you’ll need to edit the /etc/apache2/mods-enabled/dir.conf file and change the order in which the index.php file is listed within the DirectoryIndex directive:
```
sudo vim /etc/apache2/mods-enabled/dir.conf
```
```
<IfModule mod_dir.c>
        #Change this:
        #DirectoryIndex index.html index.cgi index.pl index.php index.xhtml index.htm
        #To this:
        DirectoryIndex index.php index.html index.cgi index.pl index.xhtml index.htm
</IfModule>
```
After saving and closing the file, you will need to reload Apache so the changes take effect:
```
sudo systemctl reload apache2
```
Finally, we will create a PHP script to test that PHP is correctly installed and configured on your server.

Now that you have a custom location to host your website’s files and folders, we’ll create a PHP test script to confirm that Apache is able to handle and process requests for PHP files.

Create a new file named index.php inside your custom web root folder:
```
vim /var/www/projectlamp/index.php
```
This will open a blank file. Add the following text, which is valid PHP code, inside the file:
```
<?php
phpinfo();
```
When you are finished, save and close the file, refresh the page and you will see default php page
