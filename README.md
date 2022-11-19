# Setting up your first Ubuntu VPS
This Tutorial shows you how to secure your VPS (Virtual Private Server). It is crucial to follow the best practices because you don't want to have a malicious actor in your account. 

Tested with Ubuntu 18.04.

First we want to check that our VPS host provides a console login without ssh. This is important because if we lock ouself out we lose the access to our VPS. **Warning**. Take this serious. You don't want to lock yourself out. If you mess up this configuration you can no longer access via ssh. No guarantee for this! And no guarantee for the tutorial. But it's not rocket science. 
I tested it on two different devices without conflicts.

## 1. Connecting to the VPS 
When you recieve your loggin data from the web host you usually get three important information:

* your IP address
* your username (which is in general <root>)
* your 

for <root>

### 1a. Connecting from MAC OS
Open your terminal and use the following command to connect to your VPS. You should of course insert the information you recieved from the web host.
```
ssh <username>@<IP-address>
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
That's it. You have now *full* control of your first VPS. But as useful this may occur at the beginning there is also a downside. Everybody with the password can connect via ssh to your VPS and change everything he wants. Let's change this.

### 1b. Connecting from Windows
For this tutorial we use the free SSH and telnet client - software PuTTY. You can download it on https://www.putty.org/.
Open puTTY.exe and use the following settings to connect to your VPS. Of course you should insert the information you recieved from the web host.

Host Name (or IP address):	root@178.182.442.192
Port:				22
Connection type:		SSH

Click on "Open" to connect to your VPS.
You will be asked to enter your password. After entering it and pressing ENTER you are logged in. 
That's it. You have now *full* control of your first VPS. But as useful this may occur at the beginning there is also a downside. Everybody with the password can connect via ssh to your VPS and you change everything he wants. Let's change this.


## 2 Updating Ubuntu on VPS
We should make sure that our software is up to date. Updating is important to make sure that you don't miss security updates which gives an potential attacker better chances to highjack your VPS. Let's run the update.
```
sudo apt update
sudo apt full-upgrade
```
You may wonder why we use the sudo prefix here. It is not necessary when you are logged in as <root>.
### Checking the Timezone
Choose the right timezone.
```
sudo dpkg-reconfigure tzdata
```
The menu is self-explanatory. 

## 3. Generate a new User
Everybody who knows the password can log into your VPS and has immediatelly *full* rights. 
That is because the user <root> is the superuser. Think of it as the admin of this computer. It is good practice to add a new user.
**Don't just copy this command!** Change the field <yournewuser> to something of your choice. It is just a name. But better don't use your clear name. Bots are using common names from the country to try to login into your VPS. So be creative! And use lower case letters. (The name should match the regular expression defined in the /etc/adduser.conf and is usually NAME_REGEX="^[a-z][-a-z0-9_]*\$" . If you don't understand this - just use lowercase.)
```
adduser <yournewuser>

```
It will be prompted to first enter the password of the current user and then to choose a password for <yournewuser>. Please choose a **strong password** and safe it in your password manager.(If you don't have one - go get it. You need it in crypto. But after this tutorial.) Make sure you don't loose it. There is no need to fill in the information like name, room ....just leave it blank. 

### make him a sudoer
Now we have a new user. We want to add the <yournewuser> to the sudo list because the new user has limited rights.
**Don't just copy this command!** Change the field <yournewuser> to something of your choice.
```
usermod -aG sudo <yournewuser>
```
Our new user has now sudo privileges. Now let's check if everything is ok.
```
su - yournewuser
sudo su
whoami
```
The output should be 'root' which means we have succesfully generated a new user and added him to the group of sudoers.
Now you can open a new terminal and connect to your new user via ssh. **For windows please look at storypoint 1b.**
```
ssh <yournewuser>@<IP-address>
```
You see why we need a strong password? At the moment everybody with the password of the <yournewuser> can login to your VPS. Let's work an that.

## 4. Generating a RSA key pair
Our goal in this chapter is to activate the ssh login with a private key. After that we will turn off the password login. 

### 4a.  MAC OS
Let's open a new terminal on your **local** machine. 
Navigate to your /.ssh folder and generate the RSA keys. 
```
cd ~/.ssh/
ssh-keygen -t rsa
```
You will be asked to name the file. The convention is: *id_rsa_xxx* . Where 'xxx' is something you can choose. It might be *id_rsa_coda*.
Next step is to choose a password to encrypt the private key. Choose a strong password and save the password in your password manager. Remember, if you lose your password you lose the access to your VPS.

You have successfully generated two files on your local machine.
1. the file 'id_rsa_xxx' which is your **private key** and stays on your local machine.
2. the file 'id_rsa_xxx.pub' this is your **public key** which we need to upload to your VPS.


Let's upload the file to your VPS:**Don't just copy this command!** Make sure that you don't copy it to the 'root' user. Copy it to the user we just generated. The command format looks like this.
```
ssh-copy-id [-i [id_rsa_xxx.pub]] <yournewuser>@<IP-address>
```
Let's illustrate this on an example.
```
ssh-copy-id -i ~/Users/user_on_your_local_machine/.ssh/id_rsa_coda.pub <yournewuser>@<IP-Address>
```
Now you should see something like 'number of keys added: 1' in your terminal. Perfect! We try to connect with our RSA key.
```
ssh -i ~/.ssh/id_rsa_coda <yournewuser>@<IP-address>
```
Enter your password which you used for encrypting your RSA key. (Not your user password.) Voila! 

### Organizing RSA keys - MAC OS
For every VPS you are using you will have a different RSA key (hopefully). Maybe you also use one for your git account. Therefore it is recommended to organize your keys from the beginning on. Believe me, it saves you a lot of time. So go to your ~/.ssh/ folder and look for the file named *config*. It's usually located here:

```
cd /Users/your_user_name_on_local_machine/.ssh/config
```
If this file does not exist we simply create the file in the /.ssh/ folder
```
nano config
```
Nano is like word, just a texteditor. We are going to add the following to the file: make sure you replace Host, IP-address, yournewuser, and id_rsa_xxx with your names.
```
Host <vps_coda>
  Hostname <IP-address> 
  user <yournewuser>
  IdentityFile ~/.ssh/<id_rsa_xxx>
```
You can exit Nano and save the file. Press ctrl + x and confirm to save the file. Done!
Now back to the terminal on your local machine.
```
ssh vps_coda
```
Enter the encryption password for your RSA key and you are in your VPS.


### 4b. on Windows using PuTTY
lets open puttygen.exe on your local machine to generate your RSA key pair.
the paramters should be set as following:

- Type of key to generate= RSA
- number of bits in a generated key= 2048

Click on generate. While generation you have to move your mouse over the application window for some random computations.
Once it´s done you can set a Key comment, that helps you to organize your Keys, if you have some more.
Next step is to choose a password (Key passphrase) to encrypt the private key. Choose a strong password and save the password in your password manager. Remember, if you lose your password you lose the access to your VPS.

Now you can save your private key on your local machine. Later you choose that file for authentication in PuTTY.exe
The public key you have to copy to your VPS. Copy the hole string in the text area on the top of puttygen.exe to clipboard.

On your VPS you go to directory /home/<yournewuser>/.ssh
If this directory doesn´t exists, just create it.

```
sudo nano authorized_keys
```
Paste in your public key string and save that file.

Now you should be able to login via PuTTY with your private-key. Let´s check this!
Open another putty.exe on your local machine. Go to "Connection-->SSH-->Auth on the Category-tree on the right side.
There you have to choose your stored private key file at the "Authentication parameters".
Go back to the Session settings. 

Host Name (or IP address):	<yournewuser>@178.182.442.192
Port:				22
Connection type:		SSH

Click on "Open".
Now you are asked for your Key passphrase. The password you set while generating your RSA key pair. (Not your user password.) Voila! 

### Organizing RSA keys - in PuTTY
It´s recommended to save your putty - settings by type in a descriptive name at "Saved Sessions" and click on "Save".
So later on you can open a connection by double-clicking the "Saved Sessions" settings.
Now you can open your Connection.


## 5. Disable Password login
**Warning:** only follow this chapter once you have logged in successfully with your RSA key. If you disable the password login and cannot connect with the private key then you locked yourself out. Also make sure that you have a console from your VPS provider just in case you lock yourself out.


Since you are logged in your VPS with <yournewuser> we first want to have root previliges.
```
sudo su
```
Enter user password. 
```
grep -q "^PasswordAuthentication" /etc/ssh/sshd_config && sed -i 's/^PasswordAuthentication.*/PasswordAuthentication no/g' /etc/ssh/sshd_config || echo "PasswordAuthentication no" >> /etc/ssh/sshd_config
```
This command changes the sshd_config file in the ssh folder and sets the PasswordAuthentication to 'no'. You can also do this manually with the help of nano. Just navigate to the directory, open the file sshd_config and set 'PasswordAuthentication no'.
```
systemctl restart sshd
```
Restarting the ssh daemon. Now we want to try if everything went well. Open a terminal on your locale machine and try to connect to your VPS using a password.
```
ssh root@<IP-address>
```
You should see something like: *Permission denied (publickey).*
Check this one:
```
ssh <yournewuser>@<IP-address>
```
Again you should see: *Permission denied (publickey).*
Nice! No more brute force attacks on your watch.


### Enabling password login again
You might be interested **for tests** to enable the password login again. Here you go.
Remember to use those commands with sudo.

```
sed -i 's/^PasswordAuthentication.*/PasswordAuthentication yes/g' /etc/ssh/sshd_config
systemctl restart sshd
```
But it is recommended to only allow the login with RSA keys. 

## 6. Disable root login
Every malicious actor knows that there is a user root. So let's lock the root access. You might ask yourself why we do this since the root user has no ssh key and therefore nobody can connect to it. True. Anyway i recommend doing it because of good practice. You never know. You might turn on the password login again and suddenly your user 'root' can be brute forced.
Use sudo of course ('sudo su' is the trick):

```
sed -i 's/^PermitRootLogin.*/PermitRootLogin no/g' /etc/ssh/sshd_config
systemctl restart sshd
```
This command opens the sshd_config file and sets 'PermitRootLogin no'. Maybe you should go open the file and check if it worked out. 


### enable root login
Just for tests. Here you go:
```
sed -i 's/^PermitRootLogin.*/PermitRootLogin yes/g' /etc/ssh/sshd_config
systemctl restart sshd
```

## 7. Setting up a firewall and changing ssh port
Your standard ssh port is port 22. So every bot out there is knocking on the port 22. If we change this default port we can protect ourselfs from those bots. Of course they can use a port scanner. But you know, security is nothing binary. Just make it harder for someone to crack the system.


First we want to check that our VPS host provides a console login without ssh. This is important because if we lock ouself out we lose the access to our VPS. **Warning**. Take this serious. You don't want to lock yourself out. If you mess up this configuration you no longer can access via ssh. No guarantee for this!


### Firewall
We install a firewall called ufw on your VPS as a sudoer:
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

The standard setting is that you allow all outgoing connections and deny all incoming connections.
**Warning** This is the crucial part. If we are not careful we lock ourselfs out. **Make sure you have a login to your VPS from the host provider independent of ssh.**
```
sudo ufw default deny incoming
sudo ufw default allow outgoing
```
We now want to allow the ssh connection from outside. 
```
sudo ufw allow ssh
sudo ufw allow 22/tcp
```
Restart the firewall:
```
sudo ufw disable
sudo ufw enable
sudo ufw status
```
Hopefully you will see something like '22/tcp - allow - anywhere' for IPv4 and IPv6. Good!

### Changing ssh port
You have about 65000 ports on our VPS. You should use a free port which is **not** in the range of 1-1024. I will use port 5522 for my ssh **as an example**. Some ports are reserved for specific services. (I recommend to look up a list of ports on the Internet.)

Before we change the ssh port we want to update the rules on our firewall. If we don't do this our ssh connection will be blocked by the firewall.
```
sudo ufw allow 5522/tcp
```
Check if the rules are added
```
sudo ufw status
```
Next thing is to change the ssh port.
```
sudo nano /etc/ssh/sshd_config
```
Navigate to the field port and change it to port 5522. **Please concentrate and double check** Exit and save with ctrl + x.
```
sudo systemctl restart ssh
```
You should see the ssh listening to port 5522.
```
ss -an | grep 5522
```
check also this:
```
lsof -Pni|grep sshd
```
Alright. We have updated our ufw to allow port 5522 and we have changed the ssh port. Once you leave this established session and there is a failure you will lose the access to your VPS via ssh. Don't close this terminal. Open a new one and try to connect to your VPS. Adjust the command - you know now how to do it.

```
ssh -i ~/.ssh/id_rsa_xxx <yournewuser>@IP-address -p 5522
```
This is the proof that we still have access. Nice!
Then we should close the ssh port 22 with the following command.
```
sudo ufw deny 22/tcp
```

### Updating the ssh config file on your local machine
Open the ssh config file on your local machine.
```
cd ~/.ssh/
nano config
```
Add Port 5522 to your Host - simply 'Port 5522'. Exit and save.
Connect to your VPS:
```
ssh vps_coda
```
Done!? Almost.

## 8. Update firewall to be ready for coda
As you will see while installing the coda node you will need to open port 8302 and 8303 on your VPS to be able to run the node. It's simple:
```
sudo ufw allow 8302/tcp
sudo ufw allow 8303/tcp
```
What next? Your VPS is ready to run some snarky software.
Go to:
https://codaprotocol.com/docs/getting-started

## Epilog
Run this command to see the failed login attempts on your VPS. Funny, right? What you see here are the bots which are trying to get in. But after this tutorial you shouldn't worry too much about them.
```
grep "Failed password" /var/log/auth.log
```
## Update the keys from RSA to Ed25519
Newer Ubuntu versions do not allow RSA by default and elliptic curves are simply the better public / private key encryption schemes. So let's change to ed25519.

First of all we allow the password login again on our VPS.

grep -q "^PasswordAuthentication" /etc/ssh/sshd_config && sed -i 's/^PasswordAuthentication.*/PasswordAuthentication no/g' /etc/ssh/sshd_config || echo "PasswordAuthentication no" >> /etc/ssh/sshd_config
```
This command changes the sshd_config file in the ssh folder and sets the PasswordAuthentication to 'no'. You can also do this manually with the help of nano. Just navigate to the directory, open the file sshd_config and set 'PasswordAuthentication no'.


