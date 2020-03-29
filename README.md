# VPS - Setting up your Ubuntu
This Tutorial shows you how to secure your VPS. It is crucial to follow the best practices because you don't want to have an malicious actor in your account. 

At the moment this tutorial is only for Mac OS users.

## 1. Connecting to the VPS and updating
When you recieve your loggin data from the web host you usually get three important information:
* your IP address
* your username (which is in general 'root')
* your password for 'root'

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
That's it. You have now *full* control of your first VPS. But as useful this may occur at the beginning there is also a downside. Everybody with the password can connect via ssh to your VPS and you change everything he wants. Let's change this.

### Updating Ubuntu
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



## 2. Generate a new User
Everybody who knows the password can log into your VPS and has immediatelly *full* rights. 
That is because the user 'root' is the superuser. Think of it as the admin of this computer. It is good practice to add a new user.
**Don't just copy this command!** Change the field 'yournewuser' to something of your choice. It is just a name.
```
adduser yournewuser

```
It will be prompted to first enter the password of the current user and then to choose a password for 'yournewuser'. Please choose a **strong password** and safe it in your password manager. Make sure your don't loose it. There is no need to fill in the information like name, room ....just leave it blank. 

### make him a sudoer
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
Now you can open a new terminal and connect to your new user via ssh.
```
ssh yournewuser@IP-address
```
You see why we need a strong password? At the moment everybody with the password of the yournewuser can login to your VPS. Let's work an that.

## 3. Generating a RSA key pair 
Our goal in this chapter is to activate the ssh login with a private key. After that we will turn off the password login.
Let's open a new terminal on your **local** machine. 
Navigate to your .ssh folder and run generate the RSA keys. 
```
cd ~/.ssh/
ssh-keygen -t rsa
```
You will be asked to name the file. The convention is: *id_rsa_xxx*. Where 'xxx' is something you can choose.
Next step is to choose a password to encrypt the private key. Choose a strong password and save the password in your password manager. Remember, if you lose your password you lose the access to your VPS.

You have successfully generated to files on your local machine.
1. the file 'id_rsa_xxx' which is your private key and stays on your local machine.
2. the file 'id_rsa_xxx.pub' this is your public key which we need to upload to your VPS.

Let's upload the file to your VPS:**Don't just copy this command!** Make sure that you don't copy it to the 'root' user. Copy it to the user we just generated. The command format looks like this.
```
ssh-copy-id [-i [id_rsa_xxx.pub]] [yournewuser]@IP-address
```
Let's illustrate this on an example.
```
ssh-copy-id -i ~/Users/user_on_your_local_machine/.ssh/id_rsa_coda.pub yournewuser@IP-Address
```
Now you should see something like 'number of keys added: 1' in your terminal. Perfect! We try to connect with our RSA key.
```
ssh -i ~/.ssh/id_rsa_coda yournewuser@IP-address
```
Enter your password which you used for encrypting your RSA key. (Not your user password.) Voila! 

### Organizing RSA keys
For every VPS you are using you will have a different RSA keys (hopefully). Maybe you also use one for your git account. Therefore it is recommended to organize your keys from the beginning on. Believe me, it saves you a lot of time. So go to your ~/.ssh/ folder and look for the file named *config*. It's usually located here:
```
cd /Users/your_user_name_on_local_machine/.ssh/config
```
If this file does not exist we simply create the file in the /.ssh/ folder
```
nano config
```
Nano is like word, just a texteditor. We are going to add the following now to the file: make sure you replace IP-address, yournewuser, and id_rsa_xxx with your names.
```
Host vps_coda
  Hostname IP-address 
  user yournewuser
  IdentityFile ~/.ssh/id_rsa_xxx
```
You can exit Nano and save the file. Press ctrl + x and confirm to save the file. Done!
Now back to the terminal on your local machine.
```
ssh vps_coda
```
Enter the encryption password for your RSA key and you are in your VPS.

## 4. Disable Password login
**Warning:** only follow this chapter once you have logged in successfully with your RSA key. If you disable the password login and cannot connect with the private key then you locked yourself out. 

Since you are logged in your VPS with yournewuser we first want to have root previliges.
```
sudo su
```
Enter user password. 
```
grep -q "^PasswordAuthentication" /etc/ssh/sshd_config && sed -i 's/^PasswordAuthentication.*/PasswordAuthentication no/g' /etc/ssh/sshd_config || echo "PasswordAuthentication no" >> /etc/ssh/sshd_config

```
This command changes the sshd_config file in the ssh folder and sets the PasswordAuthentication to 'no'. 
```
systemctl restart sshd
```
Restarting the ssh daemon. Now we want to try if everything went well. Open a terminal on your locale machine and try to connect to your VPS using a password.
```
ssh root@IP-address
```
You should see something like: *Permission denied (publickey).*
Check this one:
```
ssh yournewuser@IP-address
```
Again you should see: *Permission denied (publickey).*
Nice! No more brute force attacks on your watch.


### Enabling password login again
You might be interested for tests to enable the password login again. Here you go.
Remember to use those commands with sudo.

```
sed -i 's/^PasswordAuthentication.*/PasswordAuthentication yes/g' /etc/ssh/sshd_config
systemctl restart sshd
```
But it is recommended to only allow the login with RSA keys. 

## 5. Disable root login
Every malicious actor knows that there is a user root. So let's lock the root access. You might ask yourself why we do this since the root user has no ssh key and therefore nobody can connect to it. True. Anyway i recommend doing it because of good practice. You never know. You might turn on the password login again and suddenly your user 'root' can be brute forced.
Use sudo of course:
```
sed -i 's/^PermitRootLogin.*/PermitRootLogin no/g' /etc/ssh/sshd_config
systemctl restart sshd
```

### enable root login
Just for tests. Here you go:
```
sed -i 's/^PermitRootLogin.*/PermitRootLogin yes/g' /etc/ssh/sshd_config
systemctl restart sshd
```

## 6. Changing the ssh port and setting up a firewall
Your standard ssh port is port 22. So every bot out there is knocking on the port 22. If we change this default port we can protect ourselfs from those bots. Of course they can use a port scanner. But you know, security is nothing binary. Just make it harder for someone to crack the system.


First we want to check that our web host provides a console login. This is important because if we lock ouself out we lose the access to our VPS. **Warning**. Take this serious. You don't want to lock yourself out. If you mess up this configuration you no longer can access via ssh.

We install a firewall called ufw. On your VPS as a sudoer:
```
sudo apt-get install ufw

```
After the installation we want to check the status of the firewall:
```
ufw status
```
It is probably inactive. Now open the configuration file and make sure that IPv6 is activated.
```
sudo nano /etc/default/ufw
```
Check the textfile if 'IPV6' is set to 'yes' (IPV6=yes). Exit and save with ctrl + x.
```
sudo ufw disable
sudo ufw enable

The standard



