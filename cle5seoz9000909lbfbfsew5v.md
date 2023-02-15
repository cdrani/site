# DevOps PBL: DevOps Tooling Site

We want to implement a tooling website solution that makes access to DevOps tools within the corporate infrastructure easily accessible.

The tools we want our team to be able to use are well-known and widely used by multiple DevOps teams, so we will introduce a single DevOps Tooling Solution that will consist of these tools:

1. [**Jenkins**](https://www.jenkins.io/) **– free and open source automation server used to build** [**CI/CD**](https://en.wikipedia.org/wiki/CI/CD) **pipelines.**
    
2. [**Kubernetes**](https://kubernetes.io/) **– open-source container-orchestration system for automating computer application deployment, scaling, and management.**
    
3. [**Jfrog Artifactory**](https://jfrog.com/artifactory/) **– Universal Repository Manager supporting all major packaging formats, build tools and CI servers. Artifactory.**
    
4. **Rancher – an open-source software platform that enables organizations to run and manage** [**Docker**](https://en.wikipedia.org/wiki/Docker_(software)) **and Kubernetes in production.**
    
5. [**Grafana**](https://grafana.com/) **– a multi-platform open-source analytics and interactive visualization web application.**
    
6. [**Prometheus**](https://prometheus.io/) **– An open-source monitoring system with a dimensional data model, flexible query language, efficient time series database and modern alerting approach.**
    
7. [**Kibana**](https://www.elastic.co/kibana) **– Kibana is a free and open user interface that lets you visualize your** [**Elasticsearch**](https://www.elastic.co/elasticsearch/) **data and navigate the** [**Elastic Stack**](https://www.elastic.co/elastic-stack)**.**
    

In this project you will implement a solution that consists of the following components:

1. **Infrastructure**: AWS
    
2. **Webserver Linux**: RHEL 9
    
3. **Database Server**: Ubuntu 22.04 + MySQL
    
4. **Storage Server**: RHEL 9 + NFS Server
    
5. **Programming Language**: PHP
    
6. **Code Repository**: GitHub
    

In the diagram below you can see a common pattern where several stateless Web Servers share a common database and also access the same files using [Network File System (NFS)](https://en.wikipedia.org/wiki/Network_File_System) as shared file storage. Even though the NFS server might be located on completely separate hardware for Web Servers it will resemble a local file system from where they can serve the same files.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1675359347436/f9f43a9c-b5fc-472c-8e10-e7fcc5a287bc.png align="center")

It is important to know what storage solution is suitable for what use cases, thus we need to ask the following questions: what data will be stored, in what format, how this data will be accessed, by whom, from where, how frequently, etc. Based on this we will be able to choose the right storage system for your solution.

## Step 1: Prepare NFS Server

### Setup LVM on RHEL 9 OS

This setup will be very similar to the last project's LVM setup. To avoid extra work on my end of rewriting the same steps, I instead will embed the setup from the last project as a gist here. Notable changes we want to account for are\*\*:\*\*

1. Formatting the disks as `xfs` instead of `ext4`
    
2. The volume names will be `opt-lv`, `apps-lv`, and `logs-lv`
    
3. Mount points will be on `/mnt` directory for the logical volumes as follows:
    
    1. Mount `lv-apps` on `/mnt/apps` – To be used by web servers
        
    2. Mount `lv-logs` on `/mnt/logs` – To be used by web server logs
        
    3. Mount `lv-opt` on `/mnt/opt` – To be used by the Jenkins server in Next Project
        

%[https://gist.github.com/cdrani/7a69cf05152c2891ee33fa2c7b8a2902] 

Here's our setup of the logical volumes:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1675550992268/e5db7d02-6d3f-4eab-a00f-29a0d7e13beb.png align="center")

Mount the logical volumes:

```bash
sudo mkdir -p /mnt/apps /mnt/logs /mnt/opt

# syncing logs and mounting
sudo rsync -av /var/log /mnt/logs
sudo mount /dev/nfsdata-vg/logs-lv /mnt/logs
sudo rsync -av /mnt/logs/ /var/log

# mount apps-lv & opt-lv
sudo mount /dev/nfsdata-vg/apps-lv /mnt/app
sudo mount /dev/nfsdata-vg/opt-lv /mnt/opt
```

We can view our newly created mounts. We can see the **target** (`/mnt/logs`), **source** (`/dev/mapper/nfsdata-vg-logs--lv`), and **fstype (**`xfs`**).**

```bash
sudo findmnt | grep '/mnt'
```

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1675552974978/dcadf7c2-cdce-41cd-b6b7-8309102f6c9b.png align="center")

Install the NFS server, configure it to start on reboot and make sure it is up and running:

```bash
sudo yum -y update
sudo yum -y install nfs-utils
sudo systemctl enable nfs-server.service
sudo systemctl start nfs-server.service
sudo systemctl status nfs-server.service
```

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1675553473643/a929709e-e388-4dcb-97cb-62f79607a596.png align="center")

Set up permissions that will allow our Web servers to read, write and execute files on NFS.

```bash
sudo chown -R nobody: /mnt/apps
sudo chown -R nobody: /mnt/logs
sudo chown -R nobody: /mnt/opt

sudo chmod -R 777 /mnt/apps
sudo chmod -R 777 /mnt/logs
sudo chmod -R 777 /mnt/opt
```

Furthermore, we want our webservers (not created yet) to be able to access our mounts as clients. For simplicity, the webservers will all be installed within the same subnet.

We need to retrieve out `subnet cidr` value to configure access to our NFS for our web servers. The subnet value can be found within the instance `Networking` tab and following the `subnet` link:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1675563345708/105b518a-444c-4616-aa49-e45874b4ac9c.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1675563442339/26c08ab8-e4a3-4cec-bbd2-a6efd210c647.png align="center")

```bash
sudo vi /etc/exports

# /etc/exports
/mnt/apps 172.31.16.0/20(rw,sync,no_all_squash,no_root_squash)
/mnt/logs 172.31.16.0/20(rw,sync,no_all_squash,no_root_squash)
/mnt/opt  172.31.16.0/20(rw,sync,no_all_squash,no_root_squash)

# exit vi
sudo exportfs -arv
```

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1675563927023/31d9dfdf-b53b-4bf2-a1f3-5e5a10b7ed7f.png align="center")

Use `rcpinfo -p | grep nfs` to check the port used by NFS and include it as a rule in the Security Groups.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1675564214367/614b3a90-4373-4773-9ed7-498d67fd8555.png align="center")

From the above, we need to open port `2049`. Additionally, to allow access to our NFS server from clients, we also need to open **TCP 111**, **UDP 111**, and **UDP 2049.** With all these related rules to our NFS, I created a specific SG for it. The `source` field is our `subnet cidr`.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1675572200831/8098bc48-6640-4b80-97ed-28d9af0b2473.png align="center")

## Configure the Database Server

As we have done multiple times in previous projects, we want to install MySQL, create a database and user, and grant the user access to the database from the webservers `subnet cidr`:

```bash
sudo yum -y update
sudo yum -y install mysql-server

# start the mysql services
sudo systemctl enable mysqld
sudo systemctl restart mysqld
```

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1675568052266/59b0f076-9a62-4c03-befc-6d1d24257869.png align="center")

## Prepare the Web Servers

Our Web Servers need to be able to serve the same content from shared storage solutions, in our case – NFS Server and MySQL database. We have already seen how to access a MySQL server from a client. For storing shared files that our Web Servers will use – we will utilize NFS and mount previously created Logical Volume `apps-lv` to the folder where Apache stores files to be served to the users (`/var/www`).

This approach will make our Web Servers `stateless`, which means we will be able to add new ones or remove them whenever we need, and the integrity of the data (in the database and on NFS) will be preserved.

In the next steps, we will do the following 3 times:

* Launch an RHEL 9 EC2 instance
    
* Configure the NFS client (this step must be done on all three servers)
    
* Deploy a Tooling application to our Web Servers into a shared NFS folder
    
* Configure the Web Servers to work with a single MySQL database
    

We need to first install our NFS client on our Web Server instances:

```bash
sudo yum -y install nfs-utils nfs4-acl-tools
```

Then mount `/var/www/` and target the NFS server's export for \`apps\`. We should see the NFS mounted on our web server:

```bash
sudo mkdir /var/www
sudo mount -t nfs -o rw,nosuid 172.31.25.225:/mnt/apps /var/www
sudo mount -t nfs -o rw,nosuid 172.31.25.225:/mnt/logs /var/log/httpd
df -h
```

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1675584349453/9d63b5de-4b22-451b-b57e-43a090ad5c0e.png align="center")

Additionally, we can persist the above changes by adding them to our `/etc/fstab` :

```bash
# /etc/fstab
172.31.25.225:/mnt/apps /var/www nfs defaults 0 0
172.31.25.225:/mnt/logs /var/log/httpd nfs defaults 0 0
```

### Install [Remi’s repository](http://www.servermom.org/how-to-enable-remi-repo-on-centos-7-6-and-5/2790/), Apache and PHP

```bash
sudo yum -y install httpd

sudo dnf -y install https://dl.fedoraproject.org/pub/epel/epel-release-latest-9.noarch.rpm

sudo dnf -y install dnf-utils http://rpms.remirepo.net/enterprise/remi-release-9.rpm

sudo dnf -y module reset php

sudo dnf -y module enable php:remi-8.1

sudo dnf -y install php php-opcache php-gd php-curl php-mysqlnd

sudo systemctl enable php-fpm

sudo systemctl start php-fpm

sudo setsebool -P httpd_execmem 1
```

We can ascertain that our NFS is mounted properly on our web server by verifying that both `/var/www` in our Web Server(s) and `/mnt/apps` in our NFS has the same Apache files and directories. Same verification for our logs - `/var/log` for our Web Servers and \`/mnt/logs\` for our NFS

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1675573517765/9bb16be3-017b-462a-9115-7a97f8f271d3.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1675573542541/6f3e6028-4ae1-4af5-90b7-70c8d61a940d.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1675584251920/caeda0e3-64c0-43c8-8e60-390bc24b1f1f.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1675584291019/7c0e9a88-bcfa-4da5-b1f6-b6f95cfa3258.png align="center")

1. Fork the tooling source code from [Darey.io](http://Darey.io) [Github Account](https://github.com/darey-io/tooling.git) to your Github account.
    
2. Deploy the tooling website’s code to the Webserver. Ensure that the **html** folder from the repository is deployed to `/var/www/html`
    

```bash
cd /var/www/html
sudo git clone https://github.com/cdrani/tooling.git
sudo mv tooling/. .
sudo mv -R html/. .
sudo rm -R html
```

Again our web servers should have the files in `/var/www/html`:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1675585478662/045a0494-a2a0-465e-bc83-22f3e20cfd33.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1675585510807/5945e514-303c-4aa0-879b-801a4ecc6bb1.png align="center")

We need to update permissions on `/var/www/html`

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1675586165636/6cc5da7c-77d8-4a6c-a6e4-7e323974e93c.png align="center")

Update the website’s configuration to connect to the database (in `/var/www/html/functions.php` file). We will make use of the `webaccess` user with a password of `Super01!` we created earlier which has access to the \`tooling\` database:

```bash
$db = mysqli_connect('<DB Server Private IP Address>', '<MySQL username>', '<MySQL password>', '<Database>');
```

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1675626157939/5a0c2c3f-027d-4082-8814-d76f2fb74199.png align="center")

Now we need to set up a users table in our tooling database. Fortunately, we already have a pre-configured `/var/www/html/tooling-db.sql` file that will create a `users` table for us. All we have to do is import it into our `tooling` database. We can import `tooling-db.sql` script to our database with this command `mysql -h <databse-private-ip> -u <db-username> -p <db-pasword> < tooling-db.sql`.

```bash
# mysql -h <databse-private-ip> -u <db-username> -p <db-pasword> < tooling-db.sql
mysql -u webaccess -h 172.31.18.126 -p tooling < /var/www/html/tooling-db.sql
```

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1675625328790/17c6f884-a7e6-4239-8e1a-420d7d5d288a.png align="center")

```bash
sudo sed -i 's/SELINUX=enforcing/SELINUX=disabled/' /etc/sysconfig/selinux
sudo setenforce 0
sudo systemctl restart httpd
```

Visiting either of our web servers (`http://<Public IP Address>/index.php`) should display the following login page and home page upon entering our credentials:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1675626219763/fce5b45f-b7bf-4423-9e6b-5e74f4932885.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1675630363386/98f5235d-ced3-4779-9185-8cf13b33a3fb.png align="center")

## Learning Outcomes

1. Configure NFS client
    
2. Deploy a Tooling application to our Web Servers into a shared NFS folder
    
3. Configure the Web Servers to work with a single MySQL database