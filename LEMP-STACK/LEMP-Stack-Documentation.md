# WEB STACK IMPLEMENTATION (LEMP STACK) IN AWS

## Introduction

The LEMP stack is a popular open-source web development platform that consists of four main components: Linux, Nginx, MySQL, and PHP (or sometimes Perl or Python). This documentation outlines the setup, configuration, and usage of the LEMP stack.

# Step 0: Prerequisites

1. EC2 Instance of t2.micro type and Ubuntu 24.04 LTS (HVM) was launched in the us-east-1 region using the AWS console.
   ![ec2](/LEMP-STACK/Images/1.JPG)

   ![ec2](/LEMP-STACK/Images/2.JPG)

2. The security group was configured with the following inbound rules:

-  Allow traffic on port 80 (HTTP) with source from anywhere on the internet.
- Allow traffic on port 443 (HTTPS) with source from anywhere on the internet.
- Allow traffic on port 22 (SSH) with source from any IP address. This is opened by default.

    ![securitygrp](/LEMP-STACK/Images/3.JPG)

    Let's Connect our instance using SSH. This is done by using the command `cd` into the folder where the private-key was downloaded, permission was changed for the private key file using `chmod 400` then finally `ssh` into it by running the following command:

    ![SSH](/LEMP-STACK/Images/4.JPG)

    ```
    cd Downloads

    chmod 400 my-lemp-key.pem

    ssh -i "my-lemp-key.pem" ubuntu@3.80.95.227
    ```
    ![SecGrp](/LEMP-STACK/Images/5.JPG)

# Step 1 - Install Nginx Web Server

1. Update and upgrade the server’s package index

    ```
    sudo apt update
    sudo apt upgrade -y
    ```
    ![Nginx](/LEMP-STACK/Images/6.JPG)

2. Install Nginx

    ```
    sudo apt install nginx
    ```

    ![Nginx](/LEMP-STACK/Images/7.JPG)

3. Verify that Nginx is active and running as a service on Ubuntu OS.

    ```
    sudo systemctl status nginx
    ```
    If it's green and running, then Nginx is correctly installed.

    ![verify](/LEMP-STACK/Images/8.JPG)

4. The Nginx server is running and can be accessed locally in the Ubuntu shell by running the command below:

    ```
    curl http://localhost:80
    OR
    curl http://127.0.0.1:80
    ```

    ![Server](/LEMP-STACK/Images/9.JPG)

5. Test with the public IP address if the Nginx server can respond to request from the internet using the url on a browser.

    ```
    http://3.80.95.227:80
    ```
    ![IP address](/LEMP-STACK/Images/10.JPG)

    This shows that the Nginx web server is correctly installed and it is accessible through the firewall.

6. Another way to retrieve your public ip address, other than to check it in AWS web console, is to use;

    ```
    curl -s  http://3.80.95.227/latest/meta-data/public-ipv4
    ```
    The above command, gave an error 401 - Unauthorized.

    ![PublicIP](/LEMP-STACK/Images/11.JPG)

    The error "401 - unauthorized" was troubleshooted by making the following navigations from the ec2 instance page on the AWS console:

    - Actions > Instance Settings > Modify instance metadata options.

    - Then change the IMDSv2 from Required to Optional.

    ![Error](/LEMP-STACK/Images/12.JPG)

    The command was run again, this time there was no error and the public IP address displayed correctly.

    ![run](/LEMP-STACK/Images/13.JPG)

# Step 2 - Install MySQL

1. Install a relational database (RDB)

    MySQL was installed in this project. It is a popular relational database management system used within PHP environments.

    ```
    sudo apt install mysql-server
    ```

    ![RDB](/LEMP-STACK/Images/14.JPG)

2. Log in to mysql console

    ```
    sudo mysql
    ```

    ![MySQL](/LEMP-STACK/Images/15.png)

    This connects to the MySQL server as the administrative database user root infered by the use of sudo when running the command.

3.  Set a password for root user using mysql_native_password as default authentication method.

    Here, the user's password was defined as "Mayor123$"

    ```
    ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY 'Mayor123$';
    ```

    ![Rootuser](/LEMP-STACK/Images/16.JPG)

    Exit the MySQL shell.

    ```
    exit
    ```
4. Run an Interactive script to secure MySQL.

    The security script comes pre-installed with mysql. This script removes some insecure settings and lock down access to the database system.

    ```
    sudo mysql_secure_installation
    ```

    ![sudo](/LEMP-STACK/Images/17.JPG)

5. After changing root user password, log in to MySQL console.

    A command prompt for password was noticed after running the command below.

    ```
    sudo mysql -p
    ```

    ![change](/LEMP-STACK/Images/18.JPG)

    Exit MySQL shell.

    ```
    exit
    ```
# Step 3 - Install PHP

1. **Install PHP**; Nginx is installed to serve the content and MySQL is installed to store and manage data. PHP is the component of the set up that processes code to display dynamic content to the end user.

    a. Install php-fpm (PHP fastCGI process manager) and tell nginx to pass PHP requests to this software for processing. 

    b. Also, install php-mysql, a php module that allows PHP to communicate with MySQL-based databases. Core PHP packages will automatically be installed as dependencies.

    The following were installed:

    - php-fpm (PHP fastCGI process manager)
    - php-mysql

    ```
    sudo apt install php-fpm php-mysql -y
    ```


    ![php](/LEMP-STACK/Images/19.JPG)

# Step 4 - Configure Nginx to use PHP processor

1. Create a root web directory for your domain.

    ```
    sudo mkdir /var/www/projectLEMP
    ```
    ![Nginx](/LEMP-STACK/Images/20.JPG)

2. Assign the directory ownership with $USER which will reference the current system user.

    ```
    sudo chown -R $USER:$USER /var/www/projectLEMP
    ```
    ![Sudo](/LEMP-STACK/Images/21.JPG)

3.  Create a new configuration file in Nginx’s “sites-available” directory using nano.

    ```
    sudo nano /etc/nginx/sites-available/projectLEMP
     ```

    Paste in the following bare-bones configuration in the new blank file using nano text editor.

    ```
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
      ```

    ![Nginx](/LEMP-STACK/Images/22.JPG)

    Here’s what each directives and location blocks does:
    
- listen - Defines what port nginx listens on. In this case it will listen on port 80, the default port for HTTP.

- root - Defines the document root where the files served by this website are stored.

 - index - Defines in which order Nginx will prioritize the index files for this website. It is a common practice to list index.html files with a higher precedence than index.php files to allow for quickly setting up a maintenance landing page for PHP applications. You can adjust these settings to better suit your application needs.

- server_name - Defines which domain name and/or IP addresses the server block should respond for. Point this directive to your domain name or public IP address.

- location / - The first location block includes the try_files directive, which checks for the existence of files or directories matching a URI request. If Nginx cannot find the appropriate result, it will return a 404 error.

- location ~ .php$ - This location handles the actual PHP processing by pointing Nginx to the fastcgi-php.conf configuration file and the php7.4-fpm.sock file, which declares what socket is associated with php-fpm.

- location ~ /.ht - The last location block deals with .htaccess files, which Nginx does not process. By adding the deny all directive, if any .htaccess files happen to find their way into the document root, they will not be served to visitors.

    To save the file, use `ctrl + x` and then press `y` then press `enter`.

4. Activate the configuration by linking to the config file from Nginx’s sites-enabled directory.

    ```
    sudo ln -s /etc/nginx/sites-available/projectLEMP /etc/nginx/sites-enabled/
    ```

    ![alt text](23.JPG)

    This will tell Nginx to use this configuration when next it is reloaded.

5. Test the configuration for syntax error.

    ```
    sudo nginx -t
    ```

    ![alt text](24.JPG)

    If any errors are reported, go back to your configuration file to review its contents before continuing.

6. Disable the default Nginx host that is currently configured to listen on port 80 in the */sites-enabled/* directory.

    ```
    sudo unlink /etc/nginx/sites-enabled/default
    ```
     
    ![alt text](25.JPG)

7. Reload Nginx to apply the changes.

    ```
    sudo systemctl reload nginx
    ```
    ![alt text](26.JPG)

8. The new website is now active but the web root /var/www/projectLEMP is still empty. Create an index.html file in this location so to test the virtual host (server block) works as expected.

    ```
    sudo echo ‘Hello LEMP from hostname’ $(curl -s http://169.254.169.254/latest/meta-data/public-hostname) ‘with public IP’ $(curl -s http://169.254.169.254/latest/meta-data/public-ipv4) > /var/www/projectLEMP/index.html
    ```

    ![alt text](27.JPG)

9. Now go to your browser and type your server’s domain name or IP address to open the website.

    ```
    http://3.80.95.227
    ```
    ![alt text](28.JPG)

    This file can be left in place as a temporary landing page for the application until an index.php file is set up to replace it. Once this is done, remove or rename the *index.html* file from the document root as it will take precedence over *index.php* file by default.

    The LEMP stack is now fully configured. At this point, the LEMP stack is completely installed and fully operational.

    In the next step, we will create a PHP script to test that Nginx is in fact able to handle *.php* files within your newly configured website.

# Step 5 – Testing PHP with Nginx 

Test the LEMP stack to validate that Nginx can handle the .php files off to the PHP processor.

1. Create a test PHP file in the document root.

    Open a new file called info.php within the document root in your text editor.

    ```
    sudo nano /var/www/projectLEMP/info.php
    ```

    Type or paste the following lines below into the new file using nano. This is a valid PHP code that will return information about your server:

    ```
    <?php
    phpinfo();
    ```
    ![alt text](29.JPG)

2. You can now access this page in your web browser by visiting the domain name or public IP address you’ve set up in your Nginx configuration file, followed by /info.php:

    ```
    http://3.80.95.227/info.php
    ```
   ![alt text](30.JPG) 

    After checking the relevant information about the server through this page, It’s best to remove the file created as it contains sensitive information about the PHP environment and the ubuntu server. It can always be recreated if the information is needed later.

    You can use `rm` to remove that file:

    ```
    sudo rm /var/www/projectLEMP/info.php
    ```
    ![alt text](31.JPG)

# Step 6 - Retrieve Data from MySQL database with PHP

1. First, connect to the MySQL console using the root account.

    ```
    sudo mysql -p
    ```
    ![alt text](32.JPG)

2. Create a new database by running the following command from your MySQL console:

    ```
    mysql> CREATE DATABASE favoriteclub_database;
    ```

    ![alt text](image-3.png)

3. Create a new user and grant the user full privileges on the new database created.

    ```
    mysql> CREATE USER 'mayor'@'%' IDENTIFIED WITH mysql_native_password BY 'Mayor123#';

    mysql> GRANT ALL ON favoriteclub_database.* TO 'mayor'@'%';
    ```
    ![alt text](34.JPG)

4. Now exit the MySQL shell with:

    ```
    mysql> exit
    ```
5. Login to MySQL console with the custom user credentials and confirm that the new user have proper permissions (access) to favoriteclub_database.
    ```
    mysql -u mayor -p
    ```
    ![alt text](35.JPG)

6. Confirm that you have access to the **favoriteclub_database**:

    ```
    mysql> show databases;
    ```

    ![alt text](36.JPG)

7. Create a test table named todo_list. From the MySQL console, run the following statement:

    ```
     CREATE TABLE favoriteclub_database.todo_list (
     item_id INT AUTO_INCREMENT,
     content VARCHAR(255),
     PRIMARY KEY(item_id)
     );
    ```

    ![alt text](37.JPG)

8. Insert a few rows of content in the test table. You might want to repeat the next command a few times, using different values:
    ```
    mysql> INSERT INTO favoriteclub_database.todo_list (content) VALUES ("My first important item");
    mysql> INSERT INTO favoriteclub_database.todo_list (content) VALUES ("My second important item");
    mysql> INSERT INTO favoriteclub_database.todo_list (content) VALUES ("My third important item");
    mysql> INSERT INTO favoriteclub_database.todo_list (content) VALUES ("and this one more thing");
    ```

    ![alt text](38.JPG)

9. To confirm that the data was successfully saved to your table, run:

    ```
    SELECT * FROM favoriteclub_database.todo_list;
    ```
    ![alt text](39.JPG)

10. After confirming that you have valid data in your test table, you can exit the MySQL console:

    ```
    mysql> exit
    ```

## Create a PHP script that will connect to MySQL and query the content.

1. Create a new PHP file in the custom web root directory using your preferred editor. I used nano for this below:

    ```
    nano /var/www/projectLEMP/todo_list.php
    ```
    The PHP script connects to MySQL database and queries for the content of the todo_list table, displays the results in a list. If there’s a problem with the database connection, it will throw an exception.

    ```
        <?php
        $user = "mayor";
        $password = "Mayor123#";
        $database = "favoriteclub_database";
        $table = "todo_list";

        try {
        $db = new PDO("mysql:host=localhost;dbname=$database", $user, $password);
        echo "<h2>TODO</h2><ol>"; 
    foreach($db->query("SELECT content FROM $table") as $row) {
        echo "<li>" . $row['content'] . "</li>";
    }
        echo "</ol>";}   catch (PDOException $e) {
        print "Error!: " . $e->getMessage() . "<br/>";
        die();
    } 
    ```

    ![alt text](image-4.png)

    Save and close the file when you’re done editing.

2. You can now access this page in your web browser by visiting the domain name or public IP address configured for your website, followed by /todo_list.php:

    ```
    http://3.80.95.227/todo_list.php
    ```

    ![alt text](image-5.png)

    That means your PHP environment is ready to connect and interact with your MySQL server.

    ## Conclusion
    These are all the steps involved in creating a LEMP stack environment for deloying PHP websites and applications to your visitors. Developers can deploy scalable and reliable web solutions by leveraging the power of Linux, Nginx, MySQL, and PHP.
