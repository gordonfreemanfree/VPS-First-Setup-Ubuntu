# VPS-First-Setup-Ubuntu
Tutorial on how to setup a VPS running Linux.

## Connecting to the VPS
When you recieve your loggin data from the web host you usually get three important information:
* your IP address
* your username (which is in general 'root')
* your password for 'root'

### for Mac Users
Open your terminal and use the following command to connect to your VPS. You should of course insert the information you recieved from the web host.
```
ssh username@IP-address
```
Let me give you an example here:
* IP address: 178.182.442.192
* username: root
* passwword: i89maelslkejd
in this example the command will look like this:
```
ssh root@178.182.442.192
```
You will be asked to enter your password. After entering it and pressing ENTER you are logged in. 
That's it. You have now *full* control of your first VPS. But as useful this may occur at the beginning there is also a downside. 

### for Windows Users

???



## Updating Ubuntu
We should make sure that our software is up to date. Updating is important to make sure that you don't miss security updates which gives an potential attacker better chances to highjack your VPS. Let's run the update.
```
sudo apt update
sudo apt full-upgrade
```
You may wonder why we use the sudo prefix here. It is not necessary when you are logged in as 'root'.
### Checking the Timezone
Choose the right timezone.
```
sudo dpkg-reconfigure tzdata
```
The menu is self-explanatory. 



## Changing and adding a User
Everybody who knows the password can log into your VPS and has immediatelly *full* rights. 
That is because the user 'root' is the superuser. Think of it as the admin of this computer. It is good practice to add a new user.
**Don't just copy this command!** Change the field 'yournewuser' to something of your choice.
```
adduser yournewuser

```
It will be prompted to first enter the password of the current user and then to choose a password for the 'yournewuser'. Please choose a **strong password** and safe it in your password manager. Make sure your don't loose it. There is no need to fill in the information like name, room ....just leave it blank. 

Now we have a new user. We want to add the 'yournewuser' to the sudo list.
**Don't just copy this command!** Change the field 'yournewuser' to something of your choice.
```
usermod -aG sudo yournewuser
```
Our new user has now sudo privileges. Now let's check if everything is ok.
```
su - yournewuser
sudo su
whoami
```
The output should be 'root' which means we have succesfully generated a new user and added him to the group of sudoers.
Now you can open another terminal and connect to your new user via ssh.
```
ssh yournewuser@IP-address
```
You see why we need a strong password? At the moment everybody with the password of the yournewuser can login to your VPS. Let's work an that.

## Creating a new key pair 
Our goal in this chapter is to activate the ssh login with a private key. After that we will turn off the password login.
Let's open a new terminal on your local machine.
```
ssh-keygen -t rsa
```
You will be asked to name the file. The convention is: *id_rsa_xxx*. Where 'xxx' is something you can choose.
Next step is to choose a new password. Choose a strong password and save the password in your password manager. Remember, if you lose your password you lose the access to your VPS.

You have successfully generated to files on your local machine.
1. the file 'id_rsa_xxx' which is your private key and stays on your local machine.
2. the file 'id_rsa_xxx.pub' this is your public key which we need to upload to your VPS.

Let's upload the file to your VPS:**Don't just copy this command!** Make sure that you don't copy it to the 'root' user. Copy it to the user we just generated. The command format looks like this.
```
ssh-copy-id [-i [id_rsa_xxx.pub]] [yournewuser]@IP-address
```
Let's illustrate this on an example. Make sure that you don't copy it to the 'root' user. Copy it to the user we just generated.
```
ssh-copy-id -i ~/Users/user_on_your_local_machine/.ssh/id_rsa_coda.pub yournewuser@IP-Address
```
You should now see something like 'number of keys added: 1' in your terminal. Perfect! We try to connect now with our RSA key.
```
ssh -i ~/.ssh/id_rsa_coda yournewuser@IP-address
```
Enter your password which you used for encrypting your RSA key. (Not your user password.)

### Organizing keys
For every VPS you are using you will have a different RSA key. Maybe you also use one for your git account. Therefore it is recommended to organize your keys from the beginning on. Believe me, it saves you a lot of time. So go to your ~/.ssh/ folder and look for the file named *config*. It's usually located here:
```
cd /Users/your_user_name_on_local_machine/.ssh/config
```
If this file does not exist we simply create the file.
```
nano config
```
Nano is like word, just a texteditor. We are going to add the following now to the file:
```
Host vps_coda
  Hostname 123.45.67.89 
  user yournewuser
  IdentityFile ~/.ssh/id_rsa_coda
```
You can exit Nano and save the file. Press ctrl + x and confirm to save the file. Done!
Now back to the terminal. Type:
```
ssh vps_coda
```
Enter the encryption password for your RSA key and you are in your VPS.



