# DevOps PBL: WordPress Web Solution

For this project, we want to prepare storage infrastructure on two Linux servers and implement a basic web solution using [WordPress](https://en.wikipedia.org/wiki/WordPress).

1. Configure storage subsystem for Web and Database servers based on Linux OS. The focus of this part is to give you practical experience in working with disks, partitions and volumes in Linux.
    
2. Install WordPress and connect it to a remote MySQL database server. This part of the project will solidify your skills in deploying Web and DB tiers of Web solutions.
    

Three-tier Architecture is a client-server software architecture pattern that comprises 3 separate layers.

1. **Presentation Layer** (PL): This is the user interface such as the client-server or browser on your laptop.
    
2. **Business Layer** (BL): This is the backend program that implements business logic. Application or Webserver
    
3. **Data Access or Management Layer** (DAL): This is the layer for computer data storage and data access. [Database Server](https://www.computerhope.com/jargon/d/database-server.htm) or File System Server such as [FTP server](https://titanftp.com/2018/09/11/what-is-an-ftp-server/), or [NFS Server](https://searchenterprisedesktop.techtarget.com/definition/Network-File-System)
    

##### Our 3-Tier Setup

1. A Laptop or PC to serve as a client
    
2. An EC2 Linux Server as a web server (This is where we will install WordPress)
    
3. An EC2 Linux server as a database (DB) server
    

## STEP 1: Setup Web Server

We will use an RHEL9 instance as our server. Additionally, we need to create and attach three 10GB volumes to our instance.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1675399339736/299183cc-052f-4d6f-904d-fe5bebec21a2.png align="center")

The volume must be in the same Availability Zone as the instance:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1675399297199/f9a02f16-843f-4621-8ce2-17d1b1b86b55.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1675399544387/8da6a901-56c1-49d2-92a1-3dbd7a82d6b9.png align="center")

Make sure to select the correct instance for which to attach the volumes:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1675399590343/10e40461-bb86-4550-92ae-25eb4f5c5623.png align="center")

### Volume Configuration

**Use** [`lsblk`](https://man7.org/linux/man-pages/man8/lsblk.8.html) **command to inspect what block devices are attached to the server. The three newly created block devices will likely have names like** `xvdf`**,** `xvdh`**, and** `xvdg` .

```bash
lsblk
```

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1675399946562/c655be40-21f0-45a1-a5c3-4962ddf83bf1.png align="center")

We can use `df -h` to view all mounts and free space on our instance

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1675400057429/82e8fadc-0856-4ba4-be74-25f62c413e1d.png align="center")

**Use** `gdisk` utility to create a single partition on each of the 3 disks. The utility is interactive with some command options. We only need to make use of the `n` to create a new partition and `w` to save the new partition to the disk and exit.

```bash
# sudo gdisk <disk>
sudo gdisk /dev/xvdf
sudo gdisk /dev/xvdg
sudo gdisk /dev/xvdh
```

Here's an example of partitioning one of the disks using the default (pressing ENTER/RETURN key):

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1675400350275/c9376521-f703-4b4d-ae40-c7081d6975b4.png align="center")

We can now see the partitions we created in each disk now:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1675400671119/46a7a172-6734-4b84-8835-ad8f3b83c9e7.png align="center")

Run `sudo lvmdiskscan` command to check for available partitions and the type of volumes

```bash
sudo yum install -y lvm2
sudo lvmdiskscan
```

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1675401222696/f8508100-b68a-4ebb-965b-43eb490dc1db.png align="center")

Use [`pvcreate`](https://linux.die.net/man/8/pvcreate) utility to mark each of the 3 disks as physical volumes (PVs) to be used by LVM.

```bash
sudo pvcreate /dev/xvdf1
sudo pvcreate /dev/xvdf1
sudo pvcreate /dev/xvdf1
sudo pvs
```

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1675401517236/1be10449-0dbc-4592-a12c-2b38bbd6772e.png align="center")

**Use** [`vgcreate`](https://linux.die.net/man/8/vgcreate) utility to add all 3 PVs to a volume group (VG). Name the VG **webdata-vg. We can verify the creation of the VG using** `sudo vgs` :

```bash
sudo vgcreate webdata-vg /dev/xvdf1 /dev/xvdg1 /dev/xvdh1
sudo vgs
```

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1675401622500/cca8378c-5263-4ee3-9b59-03062c9ec53f.png align="center")

**Use** [`lvcreate`](https://linux.die.net/man/8/lvcreate) utility to create 2 logical volumes. **apps-lv** (***Use half of the PV size***), and **logs-lv** ***Use the remaining space of the PV size***. **NOTE**: apps-lv will be used to store data for the Website while logs-lv will be used to store data for logs.

```bash
sudo lvcreate -n apps-lv -L 14G webdata-vg
sudo lvcreate -n logs-lv -L 14G webdata-vg
sudo vgs
```

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1675401855842/60864e61-026e-4684-b6ab-4762684b1d65.png align="center")

Verify the entire setup:

```bash
sudo vgdisplay -v #view complete setup - VG, PV, and LV
sudo lsblk
```

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1675402013004/85402570-d39b-40aa-8d41-1ca20e60b6b9.png align="center")

From above we can see that `apps-lv` and `logs-lv` are of type `lvm`, but we want to reformat it to an [**ext4**](https://en.wikipedia.org/wiki/Ext4) filesystem.

```bash
sudo mkfs -t ext4 /dev/webdata-vg/apps-lv
sudo mkfs -t ext4 /dev/webdata-vg/logs-lv
```

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1675403795221/845be9dd-c390-4a0a-85f7-9d54a948de84.png align="center")

Create `/var/www/html` directory to store website files and mount them on apps-lv LV.

```bash
sudo mkdir -p /var/www/html
sudo mount /dev/webdata-vg/apps-lv /var/www/html
```

We follow the same step as above for the logs, but first, we need to back up the files in `/var/log` using [`rsync`](https://linux.die.net/man/1/rsync) before mounting it on the new destination folder:

```bash
sudo mkdir -p /home/recovery/logs
sudo rsync -av /var/log/ /home/recovery/logs/
sudo mount /dev/webdata-vg/logs-lv /var/log
sudo rsync -av /home/recovery/logs/. /var/log
```

## Update `/etc/fstab` file

The UUID of the device will be used to update the `/etc/fstab` file.

```bash
sudo blkid | grep 'webdata'
```

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1675404356463/0951c0cd-660a-4bbd-9921-3c17d8c79b47.png align="center")

The [`/etc/fstab`](https://linuxconfig.org/how-fstab-works-introduction-to-the-etc-fstab-file-on-linux) file stores static information about filesystems, their mount points and mount options. This file is read at boot time to determine the overall file system structure, and thereafter when a user executes the `mount` command to modify that structure.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1675404665584/74c843ab-b85f-428e-82ee-401b7d6d1d86.png align="center")

Test the configuration, reload the daemon, and verify the setup:

```bash
sudo mount -a
sudo systemctl daemon-reload
sudo df -h
```

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1675405168239/b5ab6c26-a5c1-44c4-88ab-e1cf6d3fbed1.png align="center")

### Install WordPress on WebServer

Install Apache and its dependencies and start the web server:

```bash
sudo yum -y update
sudo yum -y install wget httpd php php-mysqlnd php-fpm php-json

sudo systemctl enable httpd
sudo systemctl start httpd
```

Install PHP and its dependencies:

```bash
sudo yum install https://dl.fedoraproject.org/pub/epel/epel-release-latest-9.noarch.rpm
sudo yum install yum-utils http://rpms.remirepo.net/enterprise/remi-release-9.rpm

sudo yum module list php
sudo yum module reset php
sudo yum module enable php:remi-8.1

sudo yum -y install php php-opcache php-gd php-curl php-mysqlnd

sudo systemctl enable php-fpm
sudo systemctl start php-fpm

sudo setsebool -P httpd_execmem 1

sudo systemctl restart httpd
```

### Download WP

```bash
mkdir wordpress; cd wordpress;

wget http://wordpress.org/latest.tar.gz
sudo tar xzvf latest.tar.gz
rm -rf latest.tar.gz

cp wordpress/wp-config-sample.php wordpress/wp-config.php
cp -R wordpress /var/www/html/
```

### Configure SELinux Policies

```bash
sudo chown -R apache:apache /var/www/html/wordpress
sudo chcon -t httpd_sys_rw_content_t /var/www/html/wordpress -R
sudo setsebool -P httpd_can_network_connect=1
```

## Setup Database Server

Launch a second RedHat EC2 instance that will have a role – ‘DB Server’  
Repeat the same steps as for the Web Server, but instead of `apps-lv` create `db-lv` and mount it to `/db` directory instead of `/var/www/html/`.

### Install MySQL on DB Server

```bash
sudo yum -y update
sudo yum -y install mysql-server

sudo systemctl enable mysqld
sudo systemctl restart mysqld
```

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1675409363078/1d0d5938-c2e1-42f0-b98d-fb42f53bc418.png align="center")

### Configure DB to work with WordPress

Use `sudo mysql` to login as `root` and setup a database and a user with access to the database:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1675409631305/3d72a6d3-2a30-4adf-8e08-d4c958a5c987.png align="center")

### Configure WordPress to Connect to a Remote Database

For this, we need to open port **3306** on the DB Server instance's Security Group (SG). Furthermore, we should limit the source to our Web Server's private IP address:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1675409884294/f53316bb-d195-4349-afa6-887de421074a.png align="center")

Additionally, we forgot to open port **80** on our Web Server's instance. I have already set up a specific SG for web servers, but reference the first rule in the image above for how it should appear.

We should now be able to access our DB Server from our Web Server as we did in the previous project:

```bash
sudo mysql -u admin -p -h 172.31.28.148
```

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1675410553319/cef33283-1901-42e8-86bd-bcf412533141.png align="center")

### View WordPress Site

With our Web Server running we should be able to view our WordPress site using the server's public IP address: \`[http://54.245.4.149/wordpress](http://54.245.4.149/wordpress)/\`:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1675410878028/a59f78b5-bf4b-4f38-8800-f45af03c2ca7.png align="center")

However, we come across the above error. All that's required on our part is to update our `wp-config.php` file to provide details regarding our database settings. This file is located in `/var/www/html/wordpress/` . The `DB_HOST` value is our DB Server private IP address:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1675411176583/099b36ee-c4bd-4c23-8e6d-4e9144b35c05.png align="center")

After reloading the page we should enter the install flow:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1675411343699/dfd6d89f-3a20-4a46-9322-dddda85ee5c8.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1675411429847/8f39bfc4-11a4-4aa2-9780-d9cb5856d1b9.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1675411455810/0f097b04-621b-4e8b-9011-264d9b9b0a33.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1675411520150/22427c67-76c1-41d7-9491-e265b8f2964d.png align="center")

Note: Remember to Terminate the two instances and delete the volumes if no longer needed lest you incur added AWS costs.

### Learning Outcomes

1. Creating & attaching volumes to EC2 instances
    
2. Creating Partitions, Physical Volumes, Volume Groups, and Logical Volumes
    
3. Mounting
    
4. Installing WordPress and configuring Database