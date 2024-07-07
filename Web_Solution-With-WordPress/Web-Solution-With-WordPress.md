# Web Solution With WordPress

## Step 1 - Prepare a Web Server

1. Launch a RedHat (AMI) EC2 instance that serve as Web Server. Create 3 volumes in the same AZ as the web server EC2 each of 10GB and attach all 3 volumes one by one to the web server.

    ![alt text](/Web_Solution-With-WordPress/Images/wordpress1.JPG)

    ![alt text](/Web_Solution-With-WordPress/Images/wordpress2.JPG)

    ![alt text](/Web_Solution-With-WordPress/Images/wordpress3.JPG)

2. Connect to the Linux terminal to begin configuration.

    ```
    cd Downloads

    chmod 400 latestkey.pem

    ssh -i "latestkey.pem" ec2-user@3.80.29.214
    ```
    ![alt text](/Web_Solution-With-WordPress/Images/wordpress4.JPG)

3. Attach all three volumes one by one to your web-server EC2 instance.

- Go to the Elastic Block Store > Volumes section.
- Select the first volume, click Actions > Attach Volume.
- Select the server instance from the list and specify a device name (e.g., /dev/nvme1).
- Click Attach.
- Repeat the above steps to attach the other two volumes to the server instance using device names /dev/nvm2 and /dev/nvme3.

    ![alt text](/Web_Solution-With-WordPress/Images/wordpress5.JPG)


4. Use **lsblk** to inspect what block devices are attached to the server. All devices in Linux reside in /dev/ directory. Inspect with ls /dev/ and ensure all 3 newly created devices are there. Their name will likely be xvdf, xvdg and xvdh.

    ```
    lsblk
    ```
    ![alt text](/Web_Solution-With-WordPress/Images/wordpress6.JPG)

5. Use df -h to see all mounts and free space on the server.

    ```
    df -h
    ```
    ![alt text](/Web_Solution-With-WordPress/Images/wordpress7.JPG)

6. a. Use "gdisk" utility to create a single partition on each of the 3 disks.

    First disk

    ```
    sudo gdisk /dev/xvdf
    ```
    ![alt text](/Web_Solution-With-WordPress/Images/wordpress8.JPG)

    Second disk

    ```
    sudo gdisk /dev/xvdg
    ```
    ![alt text](/Web_Solution-With-WordPress/Images/wordpress9.JPG)

    Third disk

    ```
    sudo gdisk /dev/xvdh
    ```
    ![alt text](/Web_Solution-With-WordPress/Images/wordpress10.JPG)

7. Use "lsblk" utility to view the newly configured partitions on each of the 3 disks.

    ```
    lsblk
    ```
    ![alt text](/Web_Solution-With-WordPress/Images/wordpress11.JPG)

8. Install lvm package. Lvm2 is used for managing disk drives and other storage devices.

    ```
    sudo yum install lvm2 -y
    ```

    ![alt text](/Web_Solution-With-WordPress/Images/wordpress12.JPG)

9. Use pvcreate utility tool to mark each of the 3 disks as physical volumes (PVs) to be used by LVM. .

    ```
    sudo pvcreate /dev/xvdf1 /dev/xvdg1 /dev/xvdh1
    ```
    ![alt text](/Web_Solution-With-WordPress/Images/wordpress13.JPG)



10. Verify that each of the volumes have been created successfully

    ```
    sudo pvs
    ```
    ![alt text](/Web_Solution-With-WordPress/Images/wordpress14.JPG)

11. Use vgcreate utility to add all 3 PVs to a Volume Group (VG). Name the VG "webdata-vg". Then, Verify that the VG has been created successfully.

    ```
    sudo vgcreate webdata-vg /dev/xvdf1 /dev/xvdg1 /dev/xvdh1

    sudo vgs
    ```
    ![alt text](/Web_Solution-With-WordPress/Images/wordpress15.JPG)

12. Use **lvcreate** utility to create 2 logical volume, apps-lv (Use half of the PV size), and logs-lv (Use the remaining space of the PV size). Verify that the logical volumes have been created successfully.

    Note: apps-lv is used to store data for the Website while logs-lv is used to store data for logs.

    ``` 
    sudo lvcreate -n app-lv -L 14G webdata-vg
     
    sudo lvcreate -n logs-lv -L 14G webdata-vg
    ```

13. Verify that the logical volumes has been created,

    ````
    sudo lvs

    ````

    ![alt text](/Web_Solution-With-WordPress/Images/wordpress16.JPG)

14. Verify the entire setup    
    ```
    sudo vgdisplay -v   #view complete setup, VG, PV and LV
    ```
    ![alt text](/Web_Solution-With-WordPress/Images/wordpress17.JPG)

    ```
    lsblk
    ```
    ![alt text](/Web_Solution-With-WordPress/Images/wordpress18.JPG)

15.  Use mkfs.ext4 to format the logical volumes with ext4 filesystem.

```
sudo mkfs.ext4 /dev/webdata-vg/apps-lv

sudo mkfs.ext4 /dev/webdata-vg/logs-lv

 ```

 ![alt text](/Web_Solution-With-WordPress/Images/wordpress19.JPG)


16. Create /var/www/html directory to store website files and /home/recovery/logs to store backup of log data.


    ```
    sudo mkdir -p /var/www/html

    sudo mkdir -p /home/recovery/logs
    ```

    Then, mount /var/www/html on apps-lv logical volume

    ```
    sudo mount /dev/webdata-vg/apps-lv /var/www/html
    ```
    ![alt text](/Web_Solution-With-WordPress/Images/wordpress20.JPG)

17. Use rsync utility to backup all the files in the log directory /var/log into /home/recovery/logs (This is required before mounting the file system).

    ```
    sudo rsync -av /var/log /home/recovery/logs
    ```
    ![alt text](/Web_Solution-With-WordPress/Images/wordpress21.JPG)

18. Mount /var/log on logs-lv logical volume (All existing data on /var/log is deleted with this mount process which was why the data was backed up)

    ```
    sudo mount /dev/webdata-vg/logs-lv /var/log
    ```

    ![alt text](/Web_Solution-With-WordPress/Images/wordpress22.JPG)

19. Restore log file back into /var/log directory.

    ```
    sudo rsync -av /home/recovery/logs/log/ /var/log
    ```

    ![alt text](/Web_Solution-With-WordPress/Images/wordpress23.JPG)



20. Update /etc/fstab file so that the mount configuration will persist after restart of the server.

    Get the UUID of the device.

    ```
    sudo blkid
    ```
    ![alt text](/Web_Solution-With-WordPress/Images/wordpress24.JPG)

    
    Update the /etc/fstab file with the format shown inside the file using the UUID. Remember to remove the leading and ending quotes. Then, save and edit.

    ```
    sudo vi /etc/fstab
    ```

    ![alt text](/Web_Solution-With-WordPress/Images/wordpress25.JPG)

21. Test the configuration and reload daemon. Verify the setup.

    ```
    sudo mount -a   (To Test the configuration)

    sudo systemctl daemon-reload (Reload the daemon)

    df -h   (Verifies the setup)
    ```

    ![alt text](/Web_Solution-With-WordPress/Images/wordpress26.JPG)

## Step 2 - Prepare the Database Server

Launch a second RedHat EC2 instance that will have a role - DB Server. Repeat the same steps as for the Web Server, but instead of apps-lv, create dv-lv and mount it to /db directory.

1.  Create 3 volumes in the same AZ as the DB Server ec2 instance each of 10GB and attach all 3 volumes one by one to the DB Server.

    ![alt text](/Web_Solution-With-WordPress/Images/wordpress27.JPG)

    ![alt text](/Web_Solution-With-WordPress/Images/wordpress28.JPG)

    ![alt text](/Web_Solution-With-WordPress/Images/wordpress29.JPG)

2. Open up the Linux terminal to begin configuration on the database terminal.

    ![alt text](/Web_Solution-With-WordPress/Images/wordpress30.JPG)

3.  Use lsblk to inspect what block devices are attached to the server. Their name will likely be xvdf, xvdg and xvdh

    ```
    lsblk
    ```
    ![alt text](/Web_Solution-With-WordPress/Images/wordpress31.JPG)

4.  Use gdisk utility to create a single partition on each of the 3 disks.

    ```
    sudo gdisk /dev/xvdb
    ```
    ![alt text](/Web_Solution-With-WordPress/Images/wordpress32.JPG)

    
    ```
    sudo gdisk /dev/xvdc
    ```
    ![alt text](/Web_Solution-With-WordPress/Images/wordpress33.JPG)
    
    ```
    sudo gdisk /dev/xdvd
    ```

    ![alt text](/Web_Solution-With-WordPress/Images/wordpress34.JPG)

5. Install lvm package

    ```
    sudo yum install lvm2 -y
    ```
    ![alt text](/Web_Solution-With-WordPress/Images/wordpress35.JPG)

6. Use pvcreate utility to mark each of the 3 dicks as physical volumes (PVs) to be used by LVM. Also, use vgcreate utility to add all 3 PVs to a volume group (VG). Name the VG database-vg. Verify that each of the volumes and the VG have been created successfully.

    ```
    sudo pvcreate /dev/xvdb1 /dev/xvdc1 /dev/xvdd1
    sudo pvs
    ```
    ```
    sudo vgcreate database-vg /dev/xvdb1 /dev/xvdc1 /dev/xvdd1
    sudo vgs
    ```

    ![alt text](/Web_Solution-With-WordPress/Images/wordpress36.JPG)

7. . Use lvcreate utility to create a logical volume, db-lv (Use 20G of the PV size since it is the only LV to be created). Verify that the logical volumes have been created successfully.

    ```
    sudo lvcreate -n db-lv -L 20G database-vg

    
    sudo lvs
    ```

    ![alt text](/Web_Solution-With-WordPress/Images/wordpress37.JPG)

8.  Use mkfs.ext4 to format the logical volumes with ext4 filesystem and mount /db on db-lv

    ```
    sudo mkfs.ext4 /dev/database-vg/db-lv
    ```
    ```
    sudo mount /dev/database-vg/db-lv /db
    ```

    ![alt text](/Web_Solution-With-WordPress/Images/wordpress38.JPG)

    ![alt text](/Web_Solution-With-WordPress/Images/wordpress39.JPG)

9. Update /etc/fstab file so that the mount configuration will persist after restart of the server.

    Get the UUID of the device.

    ```
    sudo blkid
    ```
    ![alt text](/Web_Solution-With-WordPress/Images/wordpress40.JPG)

    Update the /etc/fstab file with the format shown inside the file using the UUID. Remember to remove the leading and ending quotes.

    ```
    sudo vi /etc/fstab
    ```
    ![alt text](/Web_Solution-With-WordPress/Images/wordpress41.JPG)


10. Test the configuration and reload daemon. Verify the setup.

    ```
    sudo mount -a  (Test the configuration)

    sudo systemctl daemon-reload (Reload Daemon)

    df -h   (Verifies the setup)
    ```
    ![alt text](/Web_Solution-With-WordPress/Images/wordpress42.JPG)

## Step 3 - Install WordPress on the Web Server EC2 Instance  
    
1. Update the repository.

    ```
    sudo yum -y update
    ```

    ![alt text](/Web_Solution-With-WordPress/Images/wordpress43.JPG)

2. Install wget, Apache and it's dependencies.

    ```
    sudo yum -y install wget httpd php php-mysqlnd php-fpm php-json
    ```
    ![alt text](/Web_Solution-With-WordPress/Images/wordpress44.JPG)

3. Enable and start Apache

    ```
    sudo systemctl enable httpd
    sudo systemctl start httpd
    ```

    ![alt text](/Web_Solution-With-WordPress/Images/wordpress45.JPG)

4. Install php and all its dependencies.

        ```
        sudo yum install https://dl.fedoraproject.org/pub/epel/epel-release-latest-8.noarch.rpm
        sudo yum install yum-utils http://rpms.remirepo.net/enterprise/remi-release-8.rpm
        sudo yum module list php sudo yum module reset php
        sudo yum module enable php:remi-7.4
        sudo yum install php php-opcache php-gd php-curl php-mysqlnd
        sudo systemctl start php-fpm
        sudo systemctl enable php-fpm 
        sudo setsebool -P httpd_execmem 1
        ```
    ![alt text](/Web_Solution-With-WordPress/Images/wordpress46.JPG)

5. Restart Apache.

    ```
    sudo systemctl restart httpd
    ```
    ![alt text](/Web_Solution-With-WordPress/Images/wordpress47.JPG)


    Test to see the default Apache page on a browser using the public IP address

    ![alt text](/Web_Solution-With-WordPress/Images/wordpress48.JPG)

5. Download WordPress.

    Download wordpress and copy wordpress content to /var/www/html

    ```
    sudo mkdir wordpress && cd wordpress
    sudo wget http://wordpress.org/latest.tar.gz
    sudo tar xzvf latest.tar.gz   
    ```

    ![alt text](/Web_Solution-With-WordPress/Images/wordpress49.JPG)

    After extraction, cd into the extracted wordpress and Copy the content of wp-config-sample.php to wp-config.php.

    ```
    cd wordpress/
    sudo cp -R wp-config-sample.php wp-config.php
    ```

    ![alt text](/Web_Solution-With-WordPress/Images/wordpress50.JPG)

    Exit from the extracted wordpress. Copy the content of the extracted wordpress to /var/www/html.

    ```
    cd ..
    sudo cp -R wordpress/. /var/www/html/
    ```

    ![alt text](/Web_Solution-With-WordPress/Images/wordpress51.JPG)
    

## Step 4 - Install MySQL on DB Server EC2 Instance

1. Update the EC2 Instance

    ```
    sudo yum update -y
    ```

    ![alt text](/Web_Solution-With-WordPress/Images/wordpress52.JPG)

2. Install MySQL Server.

    ```
    sudo yum install mysql-server -y
    ```

    ![alt text](/Web_Solution-With-WordPress/Images/wordpress53.JPG)

3. Verify that the service is up and running. If it is not running, restart the service and enable it so it will be running even after reboot.

    ```
    sudo systemctl start mysqld
    sudo systemctl enable mysqld
    sudo systemctl status mysqld
    ```

    ![alt text](/Web_Solution-With-WordPress/Images/wordpress54.JPG)

## Step 5 - Configure DB to work with WordPress

Create a DB user that is from the web server IP address. The user "wordpress" will be connecting to the database using the Web Server private IP address.
 - First , log in as root user

    ```
        sudo mysql
    ```

 -  Then create a new db
    ```
       CREATE DATABASE wordpress_db;
       ```

- Next step is to create a user that can access the database from the webserver.

    ```
    CREATE USER 'wordpress'@'172.31.25.135' IDENTIFIED BY 'Mayor123$';
    ```
- Grant all prvileges to the newly created user.

    ```
    GRANT ALL PRIVILEGES ON wordpress_db.* TO 'wordpress'@'172.31.25.135' WITH GRANT OPTION;

    FLUSH PRIVILEGES;
    ```

- Confirm DB was created

    ```
     SHOW DATABASES;
    ```

    ![alt text](/Web_Solution-With-WordPress/Images/wordpress55.JPG)

- Then, exit.

    Set the bind address

    The bind address is set to the private IP address of the DB Server for more security instead of to any IP address (0.0.0.0)

    ![alt text](/Web_Solution-With-WordPress/Images/wordpress61.JPG)



## Step 6 - Configure WordPress to connect to the remote database

1. Open MySQL port 3306 on the DB Server EC2 instance.

    For extra security, access to the DB Server is allowed only from the Web Server IP address. In the inbound rule of the DB Server (MysQL server), configure the connection to allow 'your-webserver-public-IP-address'/32.

    Install mysql server on the Web Server EC2 instance.

    ```
    sudo yum install mysql-server
    ```

    ![alt text](/Web_Solution-With-WordPress/Images/wordpress56.JPG)

    ```
    sudo systemctl start mysqld
    sudo systemctl enable mysqld
    sudo systemctl status mysqld
    ```
    ![alt text](/Web_Solution-With-WordPress/Images/wordpress57.JPG)

    Open wp-config.php file and edit the database information.

    ```
    cd /var/www/html
    sudo vi wp-config.php
    sudo systemctl restart httpd
    ```

    ![alt text](/Web_Solution-With-WordPress/Images/wordpress58.JPG)

    The private IP address of the DB Server is set as the DB_HOST because the DB Server and the Web Server resides in the same subnet which makes it possible for them to communicate directly. The private IP address is not an internet routable address.

    ![alt text](/Web_Solution-With-WordPress/Images/wordpress59.JPG)

    Disable the Apache default page.

    Here the default page can be renamed.

    ![alt text](/Web_Solution-With-WordPress/Images/wordpress60.JPG)

    Connect to the DB Server from the Web Server.

    ```
    sudo mysql -h 172.31.38.167 -u wordpress -p

    show databases;
    
    exit;
    ```

    ![alt text](/Web_Solution-With-WordPress/Images/wordpress62.JPG)

    Visit the web page again with the Web Server public IP address/wordpress. Then, install wordpress.

    ```
        your-ip-address/wordpress
    ```

    ![alt text](/Web_Solution-With-WordPress/Images/wordpress63.JPG)

    The implementation of this project is complete and WordPress is available to be used.

    
