## In this project i will prepare storage infrastructure on two Linux servers and implement a basic web solution using WordPress


I will configure storage subsystem for Web and Database servers based on Linux OS. Also demonstrate the process of working with disks, partitions and volumes in Linux.

I will install WordPress and connect it to a remote MySQL database server, while deploying Web and DB tiers of Web solution.

I will be using RedHat OS

**LAUNCH AN EC2 INSTANCE THAT WILL SERVE AS “WEB SERVER**

![webserver](./images/webserver.PNG)

**CREATING AND MOUNTING VOLUMES**

Create 3 volumes in the same AZ as your Web Server EC2, each of 10 GiB.

![Add_3_volumes](./images/create_volume.PNG)

Attach all three volumes one by one to your Web Server EC2 instance

Open up the Linux terminal to begin configuration

Use *lsblk* command to inspect what block devices are attached to the server.

`lsblk`

![Inspect_block](./images/disks_attached.PNG)

Use *df -h* command to see all mounts and free space on your server

![See_mount&free_space](./images/see_mounts%26free_space.PNG)

Use *gdisk* utility to create a single partition on each of the 3 disks

`sudo gdisk /dev/xvdh`


![gdisk_partition](./images/partitioning_disk.PNG)

Use *lsblk* utility to view the newly configured partition on each of the 3 disks

![view_partition](./images/add_partition.PNG)

Install *lvm2* package using *sudo yum install lvm2*. 

`sudo yum install lvm2`

![install_lvm2](./images/Install_lvm2.PNG)

Run *sudo lvmdiskscan* command to check for available partitions.

`sudo lvmdiskscan`

![check_available_partition](./images/check_avail_partition2.PNG)

**Note**: Previously, in Ubuntu we used *apt* command to install packages, in RedHat/CentOS a different package manager is used, so we shall use *yum* command instead.

Use *pvcreate* utility to mark each of 3 disks as physical volumes (PVs) to be used by LVM

```
sudo pvcreate /dev/xvdf1
sudo pvcreate /dev/xvdg1
sudo pvcreate /dev/xvdh1
```


![physical_volume](./images/mark%20as%20physical%20volume.PNG)

Verify that your Physical volume has been created successfully by running *sudo pvs*

![Verify_physical_volume](./images/verify%20physical%20volume.PNG)

Use *vgcreate* utility to add all 3 PVs to a volume group (VG). Name the VG **webdata-vg**

`sudo vgcreate webdata-vg /dev/xvdh1 /dev/xvdg1 /dev/xvdf1`

![create_volume_group](./images/create_volume_group.PNG)

Verify that your VG has been created successfully by running *sudo vgs*

`sudo vgs`

Use lvcreate utility to create 2 logical volumes. **apps-lv (Use half of the PV size)**, and **logs-lv Use the remaining space of the PV size. NOTE**: apps-lv will be used to store data for the Website while, logs-lv will be used to store data for logs.

```
sudo lvcreate -n apps-lv -L 14G webdata-vg
sudo lvcreate -n logs-lv -L 14G webdata-vg
```

Verify that your Logical Volume has been created successfully by running *sudo lvs*

![Verify_physical_volume](./images/verify_logical_volume.PNG)

Verify the entire setup

`sudo vgdisplay -v #view complete setup - VG, PV, and LV`

`sudo lsblk `

![Verify_entire_setup](./images/verify_entire_setup.PNG)

Use *mkfs.ext4* to format the logical volumes with *ext4* filesystem

```
sudo mkfs -t ext4 /dev/webdata-vg/apps-lv
sudo mkfs -t ext4 /dev/webdata-vg/logs-lv
```

![format_logical_volume](./images/format_logical_volume.PNG)

Create **/var/www/html** directory to store website files

`sudo mkdir -p /var/www/html`

Create **/home/recovery/logs** to store backup of log data

`sudo mkdir -p /home/recovery/logs`

Mount **/var/www/html** on *apps-lv* logical volume

`sudo mount /dev/webdata-vg/apps-lv /var/www/html/`

Use *rsync* utility to backup all the files in the log directory **/var/log** into **/home/recovery/logs** (This is required before mounting the file system)

`sudo rsync -av /var/log/. /home/recovery/logs/`

![backup_files](./images/restore_log_files22.png)

Mount **/var/log** on **logs-lv** logical volume. (Note that all the existing data on /var/log will be deleted. That is why step 15 above is very
important)

`sudo mount /dev/webdata-vg/logs-lv /var/log`

Restore log files back into **/var/log** directory

`sudo rsync -av /home/recovery/logs/. /var/log`

Update */etc/fstab* file so that the mount configuration will persist after restart of the server.

**UPDATE THE */ETC/FSTAB* FILE**

The UUID of the device will be used to update the */etc/fstab* file;

`sudo blkid`

`sudo vi /etc/fstab`

Update */etc/fstab* in this format using your own UUID and rememeber to remove the leading and ending quotes.

![update_UUID](./images/update%20UUID.PNG)

Test the configuration and reload the daemon

```
 sudo mount -a
 sudo systemctl daemon-reload
 ```


 Verify your setup by running *df -h*, output must look like this

 `df -h`

![reload_demon](./images/verify_setup_again.PNG)


 ## Prepare the Database Server

 Launch a second RedHat EC2 instance that will have a role – ‘DB Server’
 
 Repeat the same steps as for the Web Server, but instead of *apps-lv* create *db-lv* and mount it to */db* directory instead of */var/www/html/*.

 ## Install WordPress on your Web Server EC2


Update the repository

`sudo yum -y update`

![update_webserver](./images/update_webserver.PNG)

Install wget, Apache and it’s dependencies

`sudo yum -y install wget httpd php php-mysqlnd php-fpm php-json`

![install_Apache](./images/Insta_Apache%2Cwget%2Cjson.PNG)

Start Apache

```
sudo systemctl enable httpd
sudo systemctl start httpd
```
![Start_Apache](./images/start_Apache.PNG)

To install PHP and it’s depemdencies

```
sudo yum install https://dl.fedoraproject.org/pub/epel/epel-release-latest-8.noarch.rpm
sudo yum install yum-utils http://rpms.remirepo.net/enterprise/remi-release-8.rpm
sudo yum module list php
sudo yum module reset php
sudo yum module enable php:remi-7.4
sudo yum install php php-opcache php-gd php-curl php-mysqlnd
sudo systemctl start php-fpm
sudo systemctl enable php-fpm
setsebool -P httpd_execmem 1
```


![enable_httpd](./images/install_PHP.PNG)

Restart Apache

`sudo systemctl restart httpd`

![Restart_Apache](./images/restart_apache.PNG)

Download wordpress and copy wordpress to *var/www/html*

```
mkdir wordpress
cd   wordpress
sudo wget http://wordpress.org/latest.tar.gz
sudo tar xzvf latest.tar.gz
sudo rm -rf latest.tar.gz
cp wordpress/wp-config-sample.php wordpress/wp-config.php
cp -R wordpress /var/www/html/
```

![enable_httpd](./images/download_wordpress.PNG)  

Configure SELinux Policies

```
  sudo chown -R apache:apache /var/www/html/wordpress
  sudo chcon -t httpd_sys_rw_content_t /var/www/html/wordpress -R
  sudo setsebool -P httpd_can_network_connect=1
```
![Configure SELinux](./images/configure_SELinux_policy.PNG)  


## Install MySQL on your DB Server EC2

```
sudo yum update
sudo yum install mysql-server
```

![Install_Mysql](./images/install_Mysql2.PNG) 

Verify that the service is up and running by using *sudo systemctl status mysqld*, if it is not running, restart the service and enable it so it will be running even after reboot


![Install_Mysql](./images/restart%26enble_Mysql.PNG)

## Configure DB to work with WordPress

```
sudo mysql
CREATE DATABASE wordpress;
CREATE USER `myuser`@`<Web-Server-Private-IP-Address>` IDENTIFIED BY 'mypass';
GRANT ALL ON wordpress.* TO 'myuser'@'<Web-Server-Private-IP-Address>';
FLUSH PRIVILEGES;
SHOW DATABASES;
exit
```

![create_user](./images/create_user2.PNG)

## Configure WordPress to connect to remote database

**Hint**: Do not forget to open MySQL port 3306 on DB Server EC2. For extra security, you shall allow access to the DB server **ONLY** from your Web Server’s IP address, so in the Inbound Rule configuration specify source as /32

![Allow_port_3306](./images/allow_port_3306.PNG)


Install MySQL client and test that you can connect from your Web Server to your DB server by using *mysql-client*

```
sudo yum install mysql
sudo mysql -u admin -p -h <DB-Server-Private-IP-address>
```

![remote_login](./images/remote_DB_login2.PNG)

Verify if you can successfully execute *SHOW DATABASES;* command and see a list of existing databases

![remote_login](./images/test_command2.PNG)

Change permissions and configuration so Apache could use WordPress

Enable TCP port 80 in Inbound Rules configuration for your Web Server EC2 (enable from everywhere 0.0.0.0/0 or from your workstation’s IP)

Try to access from your browser the link to your WordPress *http://<Web-Server-Public-IP-Address>/wordpress/*

If you see this message – it means your WordPress has successfully connected to your remote MySQL database

![remote_login](./images/wordpress.PNG)

# Congratulations!

