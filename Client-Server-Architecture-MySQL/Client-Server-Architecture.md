# Implement a Client Server Architecture using MySQL Database Management System (DBMS)

## Step 1 - Create and configure two linux-based virtual servers (EC2 instance in AWS)

- MySQL Server
- MySQL Client

1. Two EC2 Instances of t2.micro type and Ubuntu 24.04 LTS (HVM) was launched in the us-east-1 region using the AWS console.

    MySQL Server

    ![alt text](/Client-Server-Architecture-MySQL/Images/server1.JPG)

    MySQL Client

    ![alt text](/Client-Server-Architecture-MySQL/Images/server2.JPG)

## Step 2 - On MySQL server Linux Server on EC2 instance, install MySQL Server software.

1. The private ssh key permission was changed for the private key file and then used to connect to the instance via SSH by running

    ```
    cd Downloads

    chmod 400 sql-server-key.pem

    ssh -i "sql-server-key.pem" ubuntu@34.239.130.146
    ```
    ![alt text](/Client-Server-Architecture-MySQL/Images/server3.JPG)

2. Update and upgrade Ubuntu

    ```
    sudo apt update && sudo apt upgrade -y
    ```

    ![alt text](/Client-Server-Architecture-MySQL/Images/server4.JPG)

3. Install MySQL Server software.

    ```
    sudo apt install mysql-server -y
    ```

    ![alt text](/Client-Server-Architecture-MySQL/Images/server5.JPG)

4. Enable mysql server

    ```
    sudo systemctl enable mysql
    ```

    ![alt text](/Client-Server-Architecture-MySQL/Images/server6.JPG)

## Step 3 - On MySQL client Linux Server on EC2 instance, install MySQL on EC2 Client software.

1. Connect to the instance

    ```
    ssh -i "sql-server-key.pem" ubuntu@44.212.23.85
    ```

    ![alt text](/Client-Server-Architecture-MySQL/Images/server7.JPG)

2. Update and upgrade Ubuntu

    ```
    sudo apt update && sudo apt upgrade -y
    ```

    ![alt text](/Client-Server-Architecture-MySQL/Images/server8.JPG)

3. Install MySQL Client software

    ```
    sudo apt install mysql-client -y
    ```

## Step 4 - Use MySQL server's local IP address to connect from MySQL client.

 By default, both of the EC2 virtual servers are located in the same local virtual network, so they can communicate to each other using local IP addresses. Use mysql server's local IP address to connect from mysql client. MySQL server uses TCP port 3306 by default so it has to be opened by creating a new entry in inbound rules in mysql server Security Groups. For extra security, access to mysql server by all IP addresses was not allowed, only the specific local (private) IP address (172.31.86.62/32) of mysql client was allowed.

![alt text](/Client-Server-Architecture-MySQL/Images/server9.JPG)

## Step 5 - Configure MySQL server to allow connections from remote hosts.

1. The security script of MySQL was run on mysql server by running the command:

    ```
    sudo mysql_secure_installation
    ```

    ![alt text](/Client-Server-Architecture-MySQL/Images/server10.JPG)


2. Access MySQL shell.

    ```
    sudo mysql
    ```

    ![alt text](/Client-Server-Architecture-MySQL/Images/server11.JPG)

  
3. On mysql server, create a user named client and a 

- Create a new user that can connect remotely 

    ```
        CREATE USER 'mayokun'@'%' IDENTIFIED BY 'Greatman28';
    ```

    ![alt text](/Client-Server-Architecture-MySQL/Images/server12.JPG)


- Grant all privileges

    ```
     GRANT ALL PRIVILEGES ON *.* TO 'mayokun'@'%' WITH GRANT OPTION;
     ```

 -  Flush privileges

    ```
      FLUSH PRIVILEGES;
      ```
      Then,
      
    ```
       exit 
       ```
       ![alt text](/Client-Server-Architecture-MySQL/Images/server13.JPG)

4. Now, configure MySQL server to allow connections from remote hosts.

    ```
    sudo vim /etc/mysql/mysql.conf.d/mysqld.cnf
    ```
    Locate ``` bind-address = 127.0.0.1```

    Replace ```127.0.0.1``` with ```0.0.0.0```

    ![alt text](/Client-Server-Architecture-MySQL/Images/server15.JPG)

    ![alt text](/Client-Server-Architecture-MySQL/Images/server14.JPG)

    Then save and close the file.


## Step 6 - From MySQL client linux server, connect remotely to MySQL server database without using SSH. 

On the terminal tab for mysql client, we can connect remotely to the mysql server using the mysql utility. To do this, we can run the connection code below:


```
   sudo mysql -u username -h mysqlserveripaddress -p
```

username = mayokun and mysqlserver privateipaddress = 172.31.90.177

 Then, enter your password you created earlier in the setup.

![alt text](/Client-Server-Architecture-MySQL/Images/server16.JPG)

## Step 7 -Check that the connection to the remote MySQL server was successful and  you can perform SQL queries.

```
    show databases;
```

![alt text](/Client-Server-Architecture-MySQL/Images/server17.JPG)

### Manipulating the database

I manipulated the database further. First, I created a new database and new table, inserted rows/records into the table and selected all the records from the table.

- Create a new database

    ```   
    CREATE DATABASE Delicacies;
    ```

- Select the new database to use it.

    ```
    USE Delicacies;
    ```

- Creat a table
    ```
    CREATE TABLE favoritefood (
  item_id INT AUTO_INCREMENT,
  content VARCHAR(255),
  PRIMARY KEY(item_id)
    );
    ```

- Insert data/records into the table
    ```
    INSERT INTO favoritefood (content) VALUES ("My best food is Fried rice and Chicken");
    ```
    ```
    INSERT INTO favoritefood (content) VALUES ("My second best food is Pounded yam with Egusi Soup and Turkey");
    ```
- Verify the inserted data/records

    ```
    SELECT * FROM favoritefood;
    ```

    ![alt text](/Client-Server-Architecture-MySQL/Images/server18.JPG)

    At this point, this project is successfully completed. In this project, we set up a functional client-server architecture using MySQL Database Management System on AWS EC2 instances.

