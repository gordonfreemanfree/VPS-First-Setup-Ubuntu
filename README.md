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
*Don't just copy this command!* Change the field 'yournewuser' to something of your choice.
```
adduser yournewuser

```
Now we have a new user. 
