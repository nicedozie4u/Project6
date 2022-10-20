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

