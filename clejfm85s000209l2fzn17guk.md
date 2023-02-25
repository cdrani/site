# Create User with Non-Interactive Shell

## Task

> The System admin team of `xFusionCorp Industries` has installed a backup agent tool on all app servers. As per the tool's requirements, they need to create a user with a non-interactive shell.
> 
> Therefore, create a user named `javed` with a non-interactive shell on the `App Server 2`

### Information

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1677294685541/587837be-e9f9-4f69-a1b1-f91dcd6bd99e.png align="center")

## Implementation

1. ssh into `App Server 2` using as `steve` : `ssh steve@172.16.238.11` . Enter password when prompted. Currently only users `steve` and `ansible` exist in this server
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1677295895067/997a5cc1-531c-46f0-9d32-adf2222c6c46.png align="center")
    
2. Create user `javed` with non-interactive shell: `sudo adduser -s /sbin/nologin javed` . `-s` is a flag to set the shell for a user. Below I authenticate as `root` so the `sudo` prefixes are unnecessary.
    

> **<mark>nologin</mark>** displays a message that an account is not available and closes the connection and returns non-zero. It is intended as a replacement shell field to deny login access to an account.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1677296174159/d4507bc4-38f7-421d-a7b3-d8e8800a460b.png align="center")

## Learnings

1. Creating a user with a non-default $SHELL. Th `nonlogin` shell is non-interactive and used for disabling a user account in the case of suspicious activity or upon user workplace/contract termination. The shell can also be updated using `usermod -s` .
    
2. A custom message can be set for any attempts to log in as the disabled user account by editing/creating a `/etc/nologin.txt` file