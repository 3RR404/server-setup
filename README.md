# SERVER

## Introduction
Install Chrome extension for best readin' this article.
[Mardown viewer](https://chrome.google.com/webstore/detail/markdown-viewer/ckkdlimhmcjmikdlpkmbgfkaikojcbjk)

## New subdomain installation
1. create new folder for subdomain `<subdomain-name>`  
```cli
$ mkdir /var/www/<WEB>/sub/<subdomain-name>/
```
  
1. redirect content from main domain `new-example.com`  
```cli
$ cp /etc/apache2/sites-available/example.com.conf /etc/apache2/sites-available/new-example.com
```
  - is important change `document root` in file at new-example.com
  
1. enabe site and restart server  
```cli
$ a2ensite new-example
$ systemctl reload apache2
```
---

## Add User

Let's start with 
- adding new user 
- add user to sudo group
```cli
$ sudo useradd 'username'
$ sudo usermod -aG sudo username
```
  
Let’s say you want to change the default login shell from /bin/sh to /bin/bash. To do that, specify the new shell as shown below:  
```cli  
$ sudo useradd -D -s /bin/bash
```

You can verify that the default shell value is changed by running the following command:  
```  
$ sudo useradd -D | grep -i shell  
  
Output:  
SHELL=/bin/bash  
```

---
## VSFTPD

```cli
$ sudo apt update
$ sudo apt install vsftpd
$ sudo cp /etc/vsftpd.conf /etc/vsftpd.conf.orig
```

check allowed ports
```cli
$ sudo ufw status

Output
Status: active

To                         Action      From
--                         ------      ----
OpenSSH                    ALLOW       Anywhere
OpenSSH (v6)               ALLOW       Anywhere (v6)
```

Allow ports
```cli
$ sudo ufw allow 20/tcp
$ sudo ufw allow 21/tcp
$ sudo ufw allow 990/tcp
$ sudo ufw allow 40000:50000/tcp
$ sudo ufw status
```

Add new user [here](#add-user) and create home dir workspace for FTP access
```cli
$ sudo mkdir /home/<created-user>/ftp
$ sudo chown nobody:nogroup /home/<created-user>/ftp
$ sudo chmod a-w /home/<created-user>/ftp
```

Configurate FTP access
```cli
$ sudo vi /etc/vsftpd.conf
```
> Push the **I** key for insert/type text / ctrl + C - exit from typing / type :wq for write&quit - save the changes and turn off the editor  
  
### */etc/vsftpd.conf*
```conf
# Allow anonymous FTP? (Disabled by default).
anonymous_enable=NO
#
# Uncomment this to allow local users to log in.
local_enable=YES

#
# Next, let’s enable the user to upload files by uncommenting the write_enable setting
write_enable=YES

#
# We’ll also uncomment the chroot
chroot_local_user=YES
```

add a `user_sub_token` to insert the username in our `local_root` directory path
### */etc/vsftpd.conf*
```conf
user_sub_token=$USER
local_root=/home/$USER/ftp

#
# To allow FTP access on a case-by-case basis
userlist_enable=YES
userlist_file=/etc/vsftpd.userlist
userlist_deny=NO
```

Add user to list
```cli
$ echo "<user-name>" | sudo tee -a /etc/vsftpd.userlist
```

And restart service
```cli
$ sudo systemctl restart vsftpd
```
---

## MySQL 8

Login as root
```cli
$ mysql -u root -p
```

Set password for root
```cli
mysql> UPDATE mysql.user SET authentication_string=null WHERE User='root';
mysql> flush privileges;

#
# IMPORTANT replace "your_password_here"
mysql> ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY 'your_password_here';
mysql> flush privileges;
mysql> exit
```

[reset mysql password](https://devanswers.co/how-to-reset-mysql-root-password-ubuntu/)  
  
---
## Install certbot
```cli
$ sudo add-apt-repository ppa:certbot/certbot && sudo apt install python-certbot-apache
```

[setup certificate](https://www.digitalocean.com/community/tutorials/how-to-secure-apache-with-let-s-encrypt-on-ubuntu-18-04)

---

## Where find the WEB sites folder
All data are stored in `var/` folder. All websites is good enough store to semantic folder `/var/www/`

- domain **example.com** will be in `example.com/` folder
  - web/ -- all web files of main application
  - sub/ -- sub-folders representation subdomains **test/ -> test.example.com** in this folder are files of application

---

## git

Fetch the changes
```cli
$ git fetch origin
```

Create new branch and check-out to branch
```cli
$ git checkout -b "new-branch"
```

add files for commit / commit / push to **new-branch**
```cli
#
# add all modified or created files into commit
$ git add .

#
# commit with message
$ git commit -m "Message for commit"

#
# push changes to origin branch
$ git push -u origin new-branch
```

checkout branch to *master* and pull changes, while is merged
```cli
$ git checkout master

$ git pull origin master
```

list all branches and merge to another, this case to *master*
```cli
#
# list all branches
$ git branch

#
# change branch to master
$ git checkout master

#
# merge new-branch into master branch
$ git merge "new-branch"
```

List stash, aplly stash, add to stash and remove old stashed files
```cli
#
# list the stashed files
$ git stash list

#
# set stash with message
$ git stash push -m "stash-name"

#
# apply changes to files from stash /w index 0
$ git stash apply 0
```

## install Postfix on Ubuntu 18.04
[how to](https://www.digitalocean.com/community/tutorials/how-to-install-and-configure-postfix-on-ubuntu-18-04)