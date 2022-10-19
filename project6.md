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


