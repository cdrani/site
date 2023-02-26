# Linux TimeZones Setting

## Task

> During the daily standup, it was pointed out that the timezone across `Nautilus Application Servers` in `Stratos Datacenter` doesn't match the local datacenter's timezone, which is `America/Asuncion`.

## Information

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1677294685541/587837be-e9f9-4f69-a1b1-f91dcd6bd99e.png align="left")

## Implementation

We have three App Servers in which we have to update the timezones to **America/Asuncion** if not already to it. Therefore it's just a matter of accessing each server and using **timedatectl** to update the timezone. Repeat the steps below for each server:

1. `ssh` into the App Server with login details above:
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1677438222621/53936f78-0470-44b6-b673-982a0a6a5a7f.png align="left")
    

1. View the currently set Time Zone to determine if it needs to be updated. Update if not set to `America/Asuncion`:
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1677438386205/4597837f-2949-4d52-8f39-6abde2b8727b.png align="center")
    
    Looks like it needs to be updated. We can do so using `timedatectl set-timezone ZONE` command. Reference `timedatectl --help` for other commands.
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1677438905527/ac98d503-146e-4911-a3c5-137296f782b5.png align="center")
    

## Learning Takeaways

A great command to keep in the toolbox. Never had any use for it before, but I just used this to update my timezone to `America/Edmonton` on my local ubuntu server. Granted, it won't pay any dividends, other than when running the **date** command to give the current time in **MST.**