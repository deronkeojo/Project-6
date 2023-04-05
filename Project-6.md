## WEB SOLUTION WITH WORDPRESS
## WordPress is a free and open-source content management system written in PHP and paired with MySQL or MariaDB as its backend Relational Database Management System (RDBMS).

## Three-tier Architecture
Generally, web, or mobile solutions are implemented based on what is called the Three-tier Architecture.

Three-tier Architecture is a client-server software architecture pattern that comprise of 3 separate layers.

## Presentation Layer (PL): This is the user interface such as the client server or browser on your laptop.
Business Layer (BL): This is the backend program that implements business logic. Application or Webserver
Data Access or Management Layer (DAL): This is the layer for computer data storage and data access. Database Server or File System Server such as FTP server, or NFS Server
In this project, you will have the hands-on experience that showcases Three-tier Architecture while also ensuring that the disks used to store files on the Linux servers are adequately partitioned and managed through programs such as gdisk and LVM respectively.

You will be working working with several storage and disk management concepts, to have a better understanding, watch following video:
Disk management in Linux

Note: We are gradually introducing new AWS elements into our solutions, but do not be worried if you do not fully understand AWS Cloud Services yet, there are Cloud focused projects ahead where we will get into deep details of various Cloud concepts and technologies – not only AWS, but other Cloud Service Providers as well. Your 3-Tier Setup
A Laptop or PC to serve as a client
An EC2 Linux Server as a web server (This is where you will install WordPress)
An EC2 Linux server as a database (DB) server

## Step 1 — Prepare a Web Server

## Launch an EC2 instance that will serve as "Web Server". Create 3 volumes in the same AZ as your Web Server EC2, each of 10 GiB.

![create volumes](./images/Create%20volumes.png)

## Step 1 — Attach all three volumes one by one to your Web Server EC2 instance

![Attach volumes](./images/Attach%20volumes.png)

## Step 1 — Use lsblk command to inspect what block devices are attached to the server. Notice names of your newly created devices. All devices in Linux reside in /dev/ directory. Inspect it with ls /dev/ and make sure you see all 3 newly created block devices there – their names will likely be xvdf, xvdh, xvdg.

`lsblk`


![lsblk](./images/lsblk.png)
## Use df -h command to see all mounts and free space on your server

`df -h`

![df -h](./images/df-h.png)
## Use gdisk utility to create a single partition on each of the 3 disks

`sudo gdisk /dev/xvdf`

![sudo gdisk /dev/xvdf](./images/sudo%20gdisk%20.png)
## Repeat for sudo gdisk /dev/xvdg and sudo gdisk /dev/xvdh

![sudo gdisk /dev/xvdg](./images/gdisk%202.png)

![sudo gdisk /dev/xvdh](./images/gdisk%203.png)


## Install lvm2 package using sudo yum install lvm2. Run sudo lvmdiskscan command to check for available partitions.

`sudo yum install lvm2`
![sudo yum install lvm2](./images/sudo%20yum%20install%20lvm2.png)

## Use pvcreate utility to mark each of 3 disks as physical volumes (PVs) to be used by LVM

`sudo pvcreate /dev/xvdf1 /dev/xvdg1 /dev/xvdh1`

![sudo pvcreate](./images/sudo%20pvcreate.png)

## Verify that your Physical volume has been created successfully by running sudo pvs

`sudo pvs`
![sudo pvs](./images/sudo%20pvs.png)

## Use vgcreate utility to add all 3 PVs to a volume group (VG). Name the VG webdata-vg
`sudo vgcreate webdata-vg /dev/xvdh1 /dev/xvdg1 /dev/xvdf1`
![sudo vgcreate webdata](./images/sudo%20vgcreate%20webdata.png)

## Use lvcreate utility to create 2 logical volumes. apps-lv (Use half of the PV size), and logs-lv Use the remaining space of the PV size. NOTE: apps-lv will be used to store data for the Website while, logs-lv will be used to store data for logs.
`sudo lvcreate -n apps-lv -L 14G webdata-vg
sudo lvcreate -n logs-lv -L 14G webdata-vg`
![sudo lvcreate](./images/sudo%20lvcreate.png)

## Use mkfs.ext4 to format the logical volumes with ext4 filesystem
`sudo mkfs -t ext4 /dev/webdata-vg/apps-lv
sudo mkfs -t ext4 /dev/webdata-vg/logs-lv`


## Create /var/www/html directory to store website files
`sudo mkdir -p /var/www/html`


## Create /home/recovery/logs to store backup of log data
`sudo mkdir -p /home/recovery/logs`


## Mount /var/www/html on apps-lv logical volume
`sudo mount /dev/webdata-vg/apps-lv /var/www/html/`


## Use rsync utility to backup all the files in the log directory /var/log into /home/recovery/logs (This is required before mounting the file system)
`sudo rsync -av /var/log/. /home/recovery/logs/`


## Mount /var/log on logs-lv logical volume. 
`sudo mount /dev/webdata-vg/logs-lv /var/log`


## Restore log files back into /var/log directory
`sudo rsync -av /home/recovery/logs/. /var/log`
![other codes above](./images/others.png)

