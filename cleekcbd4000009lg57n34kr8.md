# Linux File Permissions

## Task

> There are new requirements to automate a backup process that was performed manually by the `xFusionCorp Industries` system admins team earlier. To automate this task, the team has developed a new bash script [`xfusioncorp.sh`](http://xfusioncorp.sh). They have already copied the script on all required servers, however they did not make it executable on one the app server i.e `App Server 3` in `Stratos Datacenter`.
> 
> Please give executable permissions to `/tmp/xfusioncorp.sh` script on `App Server 3`. Also make sure every user can execute it.

## Information

We have access to credentials servers from the [infrastructure details](https://kodekloudhub.github.io/kodekloud-engineer/docs/projects/nautilus#infrastructure-details):

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1677001104705/14f3570e-1eed-475f-916c-dce1014c04c5.png)

## Implementation

1. Starting from a **jump\_host** we ssh into **App Server 3** as the user **banner**:
    

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1677001291105/a988a430-e8ee-4e52-8b3e-cd412f7872fe.png)

In the `/tmp` directory we can view the current permissions and ownership of the `/tmp/xfusioncorp.sh` file. We can see that it belongs to `root` user and it has not been made executable:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1677001447022/09207122-aa29-4aca-bbe8-29626e662ea6.png)

1. Make `xfusioncorp.sh` **executable** for all everyone (users, groups, other):
    

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1677001715766/0e742867-ed9b-4837-a421-6c9ebf556d38.png)

1. Make `xfusioncorp.sh` **readable.** A script can't be executed if it can't be read - at least for Bourne (again) shells I believe? Let's give `read` access to everyone and see the contents of the script.
    

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1677002625010/936c3c28-eb23-4c75-9ce5-46db26e327fc.png)

1. Execute the script:
    

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1677002732962/a5d1e518-a4b8-471f-87e2-aea5450ada32.png)

## Learning Takeaways

1. Does a script file need to be **readable**? I guess all the script files I have ever written or run have always had `read` permissions set. I guess since a script is a text file it would require this permission, while binary files would not.