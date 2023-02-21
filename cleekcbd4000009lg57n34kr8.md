# Task: Linux File Permissions

> There are new requirements to automate a backup process that was performed manually by the `xFusionCorp Industries` system admins team earlier. To automate this task, the team has developed a new bash script [`xfusioncorp.sh`](http://xfusioncorp.sh). They have already copied the script on all required servers, however they did not make it executable on one the app server i.e `App Server 3` in `Stratos Datacenter`.
> 
> Please give executable permissions to `/tmp/xfusioncorp.sh` script on `App Server 3`. Also make sure every user can execute it.

We have access to credentials servers from the [infrastructure details](https://kodekloudhub.github.io/kodekloud-engineer/docs/projects/nautilus#infrastructure-details):

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1677001104705/14f3570e-1eed-475f-916c-dce1014c04c5.png align="center")

Starting from a **jump\_host** we ssh into **App Server 3** as the user **banner**:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1677001291105/a988a430-e8ee-4e52-8b3e-cd412f7872fe.png align="center")

In the `/tmp` directory we can view the current permissions and ownership of the `/tmp/xfusioncorp.sh` file. We can see that it belongs to `root` user and it has not been made executable:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1677001447022/09207122-aa29-4aca-bbe8-29626e662ea6.png align="center")

Firstly, let's make it **executable** for all everyone (users, groups, other):

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1677001715766/0e742867-ed9b-4837-a421-6c9ebf556d38.png align="center")

Additionally, we also need to make it **readable.** A script can't be executed if it can't be read. Let's give `read` access to everyone and see the contents of the script.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1677002625010/936c3c28-eb23-4c75-9ce5-46db26e327fc.png align="center")

Finally, we can execute script:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1677002732962/a5d1e518-a4b8-471f-87e2-aea5450ada32.png align="center")

## Learning Outcomes

1. Use `chmod` to edit file permissions, such as making it readable and executable
    
2. Use `chown` to change file ownership