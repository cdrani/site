---
title: "Devops Apache Load Balancer"
datePublished: Mon Feb 06 2023 05:12:50 GMT+0000 (Coordinated Universal Time)
cuid: cle57wlvr000a0ajr8anf3psm
slug: apache-load-balancer
cover: https://cdn.hashnode.com/res/hashnode/image/stock/unsplash/DX5r6BNoWVE/upload/2a726c08686fd17f0d68e2aa503dd87f.jpeg
tags: apache, load-balancing, devops-journey, pbl

---

You might have noticed the implications of having multiple web servers from the previous project - each web server has 3 different IP addresses or DNS names. This issue will be further exacerbated if we need to add more web servers. So why do we bother with this multiple servers structure, considering they are essentially clones of each other?

The rationale for this level of redundancy is a scalability issue. Many times a post/link receives an onslaught of upvotes on a site like [HackerNews](https://news.ycombinator.com/) causing thousands to millions of visitors to the link and causing the page to crash as the server was not optimized to handle the level of traffic, thus causing slow page loads or killing the server altogether. This necessitated the need for multiple servers to handle these volumes of loads by distributing these requests across their servers. Sites like **Google** and **Reddit these types** of issues to serve millions of users every day.

**Scalability** is the property of a system to be able to handle a growing load by increasing resources. There are two types of scalability approaches to consider:

1. **vertical scaling:** Increasing the CPU and ram on our current server or configuring a more powerful server as a replacement. The limitation is that there's only so much CPU and ram that can be installed on a server.
    
2. **horizontal scaling**: Spreading the load across multiple servers. This is the more common approach as it can be applied seamlessly and almost infinitely. Additionally, this can also be automated to spin up (**scale out**) or spin down (**scale in**) servers based on CPU and Memory load monitored metrics.
    

Currently, we have 2 web servers set up from the previous project. For a user/client to access them they would need to either or both of the IP addresses. This is not too bad with our limited number of servers, but this is not feasible for sites like google where they have ***millions*** of servers.

We can alleviate clients/users from this bad UX by providing only a single point of access to reach our servers. A **Load Balancer (LB)** solves this by distributing clients' requests among multiple servers and ensuring that the load is shared in an optimized manner.

Let's update our current solution architecture with an LB sandwiched between our Client and web servers. This will allow our users to access our site using a single URL/IP Address. We will make use of only 2 web servers, but the following approach will not change regardless of the number of web servers.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1675707395046/6a922056-331f-4e1c-bb4b-a71808a2f61b.png align="center")

## Prerequisites

We already have the following infrastructure setup from the previous project:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1675707672130/f998b395-cfcc-4e09-b9df-9b8edb05752f.png align="center")

## Configure Apache As A Load Balancer

For our LB we will set up an L7 Application LB with Apache. Therefore, let's create an Ubuntu EC2 instance and open port **80** in the SG.

* insert image here
    

### Install Apache

```bash
sudo apt -y update
sudo apt -y install apache2
sudo apt -y install libxml2-dev
```

### Enable Apache modules:

```bash
sudo a2enmod rewrite proxy proxy_balancer proxy_http headers lbmethod_bytraffic

# restart apache to activate enabled modules
sudo systemctl restart apache2

sudo systemctl status apache2
```

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1675709457134/514b7cbb-735c-49ae-b591-0a095ee16eb6.png align="center")

### Configure Load Balancing

Our LB configuration is done in the `/etc/apache/sites-available/000-default.conf` file. Here we can define the Web Servers and the balancing method. There are several choices for load balancing methods: [`bytraffic`](https://httpd.apache.org/docs/2.4/mod/mod_lbmethod_bytraffic.html), [`bybusyness`](https://httpd.apache.org/docs/2.4/mod/mod_lbmethod_bybusyness.html)**,** [`byrequests`](https://httpd.apache.org/docs/2.4/mod/mod_lbmethod_byrequests.html)**,** and by [`heartbeat`](https://httpd.apache.org/docs/2.4/mod/mod_lbmethod_heartbeat.html) . We can control in which proportion the traffic must be distributed by `loadfactor` parameter. Setting the same `loadfactor` to both servers will evenly distribute the traffic. I made the following changes to the file with the Private IP address of my Web Servers:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1675710694661/ee1f6279-481d-4528-94b4-5afea9ca8f6c.png align="center")

After the above changes to our config file, we need to restart our Apache server. Following that, we should be able to now visit our site using the LB's Public IP Address: \`[http://54.185.27.126/index.php](http://54.185.27.126/login.php)\`. We should be able to log in and see the same admin page from the previous project.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1675711251969/338394fa-abf7-4d2a-bfcf-d1b2e474d88c.png align="center")

We can view our Apache logs by running the following in both Web Server instances:

```bash
sudo tail -f /var/log/httpd/access_log
```

We can see below that the logs are the same - the same requests at the same times. This is due to them both sharing the same mount point `/var/log/httpd` on our NFS server. We can't differentiate between which Web Server is being selected by our LB to serve.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1675711699286/316b1cf4-5948-4230-9158-e58891d55212.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1675711735980/d5bacf23-b0e7-4ec6-b898-d8dbf5ef8f70.png align="center")

To resolve this we need to unmount our Web Servers from the NFS Server so that the Web Servers have their separate logs. Let's do the following for both Web Servers:

```bash
sudo systemctl stop httpd
sudo df -h
sudo umount /var/log/httpd
sudo df -h
```

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1675714742959/fa5a5814-c96e-4be5-a8bd-ebd2a952e578.png align="center")

Additionally, we need to prevent the `/var/log/http` directory from being mounted on boot as we currently have it configured in the `/etc/fstab` file. Let's comment it out on both servers:

```bash
#/etc/fstab

172.31.25.225:/mnt/apps /var/www nfs defaults 0 0
# 172.31.25.225:/mnt/logs /var/log/httpd nfs defaults 0 0
```

Then reload our mounts so these changes apply on boot as well and restart our Apache server:

```bash
sudo mount -a
sudo systemctl start httpd
```

Now visiting our site via our LB IP address we can view Web Server is running our requests. Below we see that our requests are being distributed equally between our two Web Servers on every page reload:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1675717647606/f3ae9d67-58e0-4a72-a082-1108658684f0.gif align="center")

This is our final setup now:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1675718156670/a33894dc-9796-4dc1-bc82-fba10e8ea7f3.png align="center")

## Learning Outcomes:

1. What is scalability? What are vertical and horizontal scaling?
    
2. What is Load Balancing? What is the purpose?
    
3. How to set up an Apache LB (install + modules + configuration with Web Servers)
    
4. View Web Servers logs