*[last update: 2013-09-27]*

# PHP gitserver #

### What is it ###
This is a script to control access to git repositories shared among many users.
It works through SSH and a single linux user at the server machine (github like).


### Definitions ###
 - `~/.git_control/git_user_permission`: user permission configuration file in JSON format. It must have `0600` mode
 - `~/.git_control/log`: log of all user requests. Logs are active by default, but they can be disabled in the `gitserver` file.


### How to use ###
Create a dedicated linux user in the server machine and lock the linux user to disable password logins using the command:
    
    passwd -l {gitlinuxuser}
        
That prevents SSH logins without a key listed in `authorized_keys`.

Create/place the repositories under this user home folder. Repository folder names are important because they will be used inside the permissions config file.

>e.g.: `~/myrepo` - means that to give access to a user in `.git_user_permission`,
the repository key must be `"myrepo"`.

Remeber that in order to create a repository as a "server" (without a working branch) you'll need to run the command init with `--bare` option.

    e.g.: git init --bare ~/myrepo
    
If yopu're creating the repository using another linux root user, it'll be necessary to set the "git" linux user as the owner of the files:

    chown -hR {gitlinuxuser}:{gitlinuxgroup} /home/{gitlinuxuser}/myrepo

### Configuration Syntax ###
The configuration is a JSON having the first key level as the usernames and has to be placed in the "git" linux user home folder: `~/.git_control/git_user_permission`. The usernames defined here are going to be used inside the `authorized_keys` file as parameters of the associated command, as will be discussed later. So make sure they match. 

Each "{username}" should point to an object having key-value pairs in which the keys are the repository folder names and the values are strings containing one of the following values: 
 - `"r"` - to give read permission,
 - `"w"` - to give write permission,
 - `"rw"` - to give read and write permissions for the associated repository. 
 
If the repository is not listed under a user, that means that the user won't have access to it. (Note: the username was picked as the top hierarchy to make it easier to know what repositories the user has acces to, and to facilitate removal of a user from the server altogether)

Configuration file example:
```json
{
    "user": {
        "reponame": "rw",
        "anotherrepo": "r"
    },

    "user2": {
        "reponame": "r"
    }
}
```


    
### Installation Steps ###
1. Check if php is installed by running:

        php -v

    if it isn't, run:

        [rpm] yum install php
        [deb] apt-get install php
        [mac] brew install php

1. Copy or move the gitserver file to `/usr/local/bin` and make it executable:

        cp gitserver /usr/local/bin/gitserver
        chmod 0755 /usr/local/bin/gitserver

1. Create a linux user, if you haven't done that yet:

        useradd {gitlinuxuser}
        
1. Create a directory `.git_control` under the home direcotry of the just created linux user. (You can configure this path in the gitserver file if necessary)

1. Create a file named `git_user_permission` under the previously created `.git_control` directory and change its mode to `0600`.

        chmod 0600 git_user_permission

1. Open `~/.ssh/authorized_keys` and add the public key for the consumer you want to grant access to by adding an entry like so:

        command="gitserver {USER}",no-port-forwarding,no-X11-forwarding,no-agent-forwarding ssh-[...]

   The `{USER}` should be replaced by the username that is going to be used inside the `.git_user_permission`, and the `ssh-[...]` should be the public key associated with that username.

1. Lock the linux user to disable password logins, by using the command:
    
        passwd -l {gitlinuxuser}

### Accessing Repos ###
1. To clone the repository in another machine, run the command:

        git clone ssh://{gitlinuxuser}@{machine-dns-or-ip}/~/myrepo
        
    or if port is required:
    
        git clone ssh://{gitlinuxuser}@{machine-dns-or-ip}:{port-number}/~/myrepo

That's it. You can now have your own lan server machine serving as many repos as necessary with user control. 
