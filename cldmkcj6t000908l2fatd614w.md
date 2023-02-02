# DevOps PBL: Onboard Users

In this project, we want to onboard 20 new Linux users onto a server. Create a shell script that reads a `CSV` file of a list of users to be onboarded. Each user requires the following:

* Must have a default `/home` directory, ie `/home/cdrani/` for user `cdrani`
    
* Each user must be added to a `developers` group
    
* An `.ssh` folder with an `authorized_keys` file that includes the `rsa.pub` key of the current user ($USER)
    

### STEP 1: Setup Private & Public keys for $USER

1. Generating ssh private (`id_rsa`) and public key (`id_rsa.pb`) for $USER. The public key will be copied over to each onboarded user's `.ssh/authorized_keys` file. We will make use of `ssh-keygen` and just press `Enter/Return` key to accept the default file names and placement. In the end, we should have both `id_rsa` and `id_rsa.pub` keys in `~/.ssh` folder.
    
    ```bash
    sudo ssh-keygen
    ```
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1675276274598/d2c35830-467f-4d35-834c-187feac40d42.png align="center")
    

### STEP 2: Onboarding Script

Now for the script itself. It's heavily documented with comments, but some additional criteria I included are to verify that the script is run by a `root` user, a `.csv` file is passed as an argument and a default password of `password` is set for each onboarded user with an expiration date of 7 days.

```bash
#!/bin/bash

# Exit if script is not run as root
if [ "$(id -u)" != "0" ]; then
  echo "This script must be run as root."
  exit 1
fi

# Exit if file is not provided
if [ $# -eq 0 ]; then
  echo "Please provide the name of the CSV file."
  exit 1
fi

# Exit if file provided is not a .csv file
file="$1"
if [ "${file##*.}" != "csv" ]; then
  echo "The file must be a CSV file."
  exit 1
fi

# Create developers group if it doesn't exist
if ! getent group developers > /dev/null 2>&1; then
  groupadd developers
fi

# Read the CSV file
while IFS=',' read -r name
do
  username=$name

  # Check if user exists
  if id "$username" >/dev/null 2>&1; then
    echo "User $username already exists."
    continue
  fi

  # Create user
  useradd "$username" -g developers -m
  echo "$username:default_password" | chpasswd

  # Set password expiration to 7 days
  # After expiration user needs to set/change password
  chage -m 7 "$username"

  # Ensure .ssh folder exists for user, otherwise create one
  home_dir=$(eval echo "~$username")
  ssh_dir="$home_dir/.ssh"
  if [ ! -d "$ssh_dir" ]; then
    mkdir "$ssh_dir"
    chmod 700 "$ssh_dir" $username
  fi

  # Add public key to users' ~/.ssh/authorized_keys
  auth_keys="$ssh_dir/authorized_keys"
  touch "$auth_keys"
  chmod 600 "$auth_keys" $username
  # Need to give ownership to user else cannot login as them!!!
  chown $username:developers "$auth_keys"
  echo $(cat ~/.ssh/id_rsa.pub) >> "$auth_key"
done < "$file"
```

To run the script we need a `*.csv` file with a list of names:

```plaintext
# names.csv
beth
mark,
lucy,
fred,
liza
```

Then we need our script to be executable before we run it:

```bash
sudo chmod +x onboard_users.sh
sudo ./onboard_users.sh names.csv
```

Let's test out some of the conditionals we have in place:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1675276127609/0250f382-5c23-4fc9-810c-178b5bb01a06.png align="center")

We now should have `/home/$user` (for ex. `/home/beth/`) directories for each user in the `names.csv` file:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1675277842824/e2d663bd-cfcb-4e5b-bcb9-2ca8eb98e374.png align="center")

Finally, we should be able to log in as any of the newly onboarded users, however, our new users don't use the same private key that we set up when launching our ec2 instance - we must use the one we generated above (`id_rsa`).

First, we need to ensure our key is not publicly viewable, i.e it is not writable:

```bash
sudo chmod -w id_rsa
```

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1675279320610/eade7910-dea6-4a81-8b11-c4927bf92235.png align="center")

After the above, we can then copy the private key from our remote server (ec2) to our local environment using `scp`:

```bash
# scp -i <key> <remote_username>@<Host>:<PathToFile>   <LocalFileLocation>
scp -i ~/Desktop/webserver-ec2.pem ubuntu@35.85.56.192:~/.ssh/id_rsa ~/.ssh/aux_id_rsa
```

With that complete, from our local computer, we can log in as an onboarded user:

```bash
ssh -i ~/.ssh/aux_id_rsa fred@38.85.56.192
```

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1675308445380/f6c00995-abd7-4175-9e20-1e161fdb96a8.png align="center")

Only users with the auth key can log in:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1675309204647/3544ed25-64df-4bf1-81c7-cc2dfe122e70.png align="center")

Note: One can log in using the `*.pem` key for the ec2 instance as well.

### Bonus:

Testing the script several times to address issues necessitated removing users as well. This quickly turned into a pain point that required a `deboard.sh` script. It has the same checks as the `onboard.sh` one which could be refactored out into a separate script that runs before both of them, but it's good enough.

```bash
#!/bin/bash

if [ "$(id -u)" != "0" ]; then
  echo "This script must be run as root."
  exit 1
fi

# Check if argument is provided
if [ $# -eq 0 ]; then
  echo "Please provide the name of the CSV file."
  exit 1
fi

# Check if argument is a .csv file
file="$1"
if [ "${file##*.}" != "csv" ]; then
  echo "The file must be a CSV file."
  exit 1
fi

# Read the CSV file
while IFS=',' read -r fname lname
do
  username=$fname

  # Check if user exists
  if ! id "$username" >/dev/null 2>&1; then
    echo "User $username does not exist."
    continue
  fi

  # Remove user
  sudo userdel -r "$username"
done < "$file"
```