# Setting up a Jupyter Lab Server for Remote Access, For Data Scientists 

There are a variety of reasons to set up a server for remote access running Jupyter lab, personally I have an older low powered laptop and want to be able to access the power of my desktop computer while I’m on the road. Setting up a server is much more of a devops task than data science, and involves a lot of research to set up the appropriate security. This guide is designed based on a Mac Mini as the host server and a MacBook Air on the client side. The command line portions of these instructions should also work on linux, however they may be more complicated than needed because of MacOS's additional security. 

1. Configuring security
    1. Creating account
    2. Allowing Access
    3. Changing SSH port 
    3. Enable Key login
    4. Disable Password login
3. Install anaconda navigator
4. Set up and test connections
5. Create bash scripting shortcuts

The documentation for Jupyter notebooks has a page on setting up access to your Jupyter notebook from the local network. That guide is available here: https://jupyter-notebook.readthedocs.io/en/stable/public_server.html. These instructions also work for JupyterLab. For the function of using an SSH tunnel I have chosen not to configure these settings. 

## VPN
Using a VPN is one of the most basic security steps you can take as an internet browser to protect your information online. Unfortunately many VPN providers block SSH traffic over their network meaning that you can’t have either your host or client device running one. 

Alternately, you can set up a personal VPN to run off the same host computer as your Jupyter server. This is helpful because it means that your server will stay only accessible via your personal VPN and not the wider internet. If you opt for this method it is possible to avoid using SSH altogether. Setting up a VPN is somewhat complicated and comes with its own set of drawbacks and I’ll address them in a separate article. 

## Configuring security
Security is an ever-changing landscape and a written article cannot keep up. Enabling access to a computer over the internet comes with inherent risks and anyone following these steps should do their research about the latest developments in security. There are no guarantees with any levels of security. 

### Create a new account
On macOS when you enable SSH access you will also be asked who can access it. I don’t use a hyper-secure password for my main admin account locally so I opted to create a separate user account that is only accessed via SSH. This account should use a password that is generated using Keychain (or another password manager). The only downside to this method is that you won’t have access to your data from the main account. It is possible to give the second account access to all this data but doing so defeats the purpose of creating an isolated account. 
To create a new account open System Preferences, and click ‘Users and Groups

![Stystem Preferences](img/2020-1-20-JupyterLab-SSH/KISIEE.png)

Click the lock icon and enter your user password

![Users & Groups](Images/Users&Groups.jpeg)

![Password Dialogue](Images/SystemPreferencesistryingtounlockUsers.jpeg)

Click the plus icon to add an account

![Users & Groups +](Images/Users&Groups.png)

By default, the new account is set up as a standard user. For security, it is best to leave this setting so that if a malicious entity gains access to your system they won’t have administrator access. Enter a name for the account (I chose remote) and then click the key icon to generate a secure password. 

![Add User](Images/NewAccountStandard.jpeg)

![Password Assistant](Images/PasswordAssistant.jpeg)

Letters and numbers or random should both create a strong password, personally, I also cranked the length up to about 20 as well just for the additional protection. Brute force is a common attack on an SSH server, so you want a strong password. Even though we will be disabling password access the additional security is recommended for access during setup. Save this password in your secure method of choice. 

Given that this account will only be accessed via SSH you can also now hide the user account to disable access via the GUI login window, this is done using the terminal command:

`sudo dscl . create /Users/remote IsHidden 1`

(Replace remote with the name of the user account you want to use) 
You will be prompted to enter a password, enter your current user account password (assuming you have administrator privileges to the computer). No text will appear in the terminal, but it is taking your password, this is expected behavior. This results in an ‘other’ user being shown on the login window. This can also be hidden using the command: 

`sudo defaults write /Library/Preferences/com.apple.loginwindow SHOWOTHERUSERS_MANAGED -bool FALSE`

It is possible to go further and also hide the user account folder but I decided I want some trace left that there is another account on my machine. Apple conveniently has created a support article which walks you through fully hiding a user account: https://support.apple.com/en-us/HT203998

### Allowing SSH Access and Remote Connecting
To allow an SSH connection on the host computer open system preferences and click sharing

![System Preferences Sharing](Images/KISTES.jpeg)

Check the box labeled ‘Remote Login’

![Sharing](Images/EEEE.jpeg)

Make sure the radio button for ‘Only these users’ is selected, then click the minus button to remove the group ‘Administrators’ followed by the plus button to add your remote user. 

![Add Remote](Images/EEEE+-.jpeg)

You can now access this computer via SSH on your local network, from your client open terminal and type:

`ssh -p [your port] remote@[yourIP]` 

Where you replace `[your port]` with the port number that you chose to listen on, and `[your ip]` is the IP of the host computer. There are serval ways to go about getting the host ID, my favorite is to go to whatsmyip.org and it will display it for you. The first time that you remote connect to a new server from your client you will be presented with the message 

`The ECDSA fingerprint is [a long alphanumeric key] Are you sure you want to continue connecting (yes/no)` 

you will need to type in yes and hit enter. It will ask for your password, which is the password to the remote account on the host computer.  

To access the computer from the web, you’ll also need to enable the port forwarding on your router. Depending on your router manufacture this process will vary. It tends to involve logging into your router's main administration page, going to the attached devices, looking up the IP address of your host computer, then going to advanced settings and port forwards to set up the port that you chose earlier to forward to your host computer. It is also possible to set up the host computer in the DMZ on some routers, this will open all ports and is not recommended for security reasons. Some routers have preset settings for certain applications and port forwards because we have set up a custom SSH port enabling the default SSH profile most likely will not work unless you can also customize which port it opens. At this point, any time that the ‘remote login’ is enabled your computer is accessible via the web. I keep mine disabled unless I’m going to use it. 


### Change the port SSH listens on
SSH is a fairly old protocol, and the port that it listens on by default is well known. There are malicious entities on the internet which set up software to automatically ‘knock’ on open ports in an attempt to find open SSH servers to brute force. Changing the port makes it harder to find your server. There are a few ways to do this, I had the most luck with editing ssh.plist. The downside to editing this plist is that you have to disable system integrity protection (SIP). For more information about what SIP is and why it exists, Apple has a good support article:https://support.apple.com/en-us/HT204899 

To disable SIP, restart your computer in recovery mode (hold cmd and r while starting your computer until the loading bar appears on screen) click ‘Utilities’ and then ‘Terminal’ and type in: 

`csrutil disable` 

It should print a confirmation message that SIP has been disabled, restart the computer into normal mode. When finished, to re-enable SIP repeat the steps above and type: 

`csrutil enable`

To edit the plist enter the following command in terminal (I like nano, but you can replace it with your text editor of choice) 

`sudo nano /System/Library/LaunchDaemons/ssh.plist` 

Again enter your administrator password when prompted. In the plist you are looking for the following lines:

![Key Sockets](Images/keySocketskey.png)

Replace both ‘ssh’ and ‘sftp-ssh’ with the number of the port you want to use, for example if you want to set it to use port 2222 you would set it to read: 

![Changed Sockets](Images/keySocketskey2222.png)


Exit the nano editor using control + x (yes control on Mac). You will be prompted asking if you want to save changes, enter y for yes. You will then be prompted to enter a name for the file, just hit enter to leave the name unchanged. At this point, I recommend restarting your computer and re-enabling SIP by reversing the steps from before. 

### Enabling key only access to your  server 
This is one of the few times where security and convenience go hand in hand. It is possible to configure your SSH server to only be accessible by ‘known hosts’. This process disables brute-forcing the password and makes it so that the greatest security vulnerability is if you ever lose control of the ‘private key’ on your machine. 

In a terminal window on your client computer type

`ssh-keygen`

This will initiate creating the key, there are a few questions asked. The first is where to save the key, you can hit enter to save it in the current directory. It will then ask you for a passphrase, this passphrase is to access the private key. This passphrase protects you in case someone gains access to your client device they will not be able to connect to your host, however, it comes at the cost of needing to enter your passphrase every time you use the private key. It then asks to confirm the passphrase, and finally displays the locations of the keys. It will also display the key ‘fingerprint’ and ‘random art’. Neither of these are terribly important and you can ignore them. 

Use `cat id_rsa.pub` in the same terminal to read the public key. You will need to copy this key onto the server. Open a new terminal and connect to your host. Enter the command

`nano ~/.ssh/authorized_keys` 

To open the nano editor. If this is the first time you’re doing this the file will likely be empty, if you have previously configured a key it will be listed here. You can now copy and paste the key that was displayed after `cat id_rsa.pub` in the first terminal window. Use control + x to exit, y to save and enter to keep the file name and location. At this point, you should be able to exit the SSH session and reconnect it, this time no password prompt should appear. 

Each user account on a computer will have its own key pair. I recommend setting up your primary user account on the host computer to also be able to ssh into the ‘remote’ account. Follow the same steps as above, and just enter the second key on a new line in 

### Disabling password access 
To complete the setup of the key only access we need to disable password access. On the host computer open terminal and type 

`sudo nano /etc/ssh/ssh_config` 

Enter your password if prompted. This will open one of the configuration files for SSH. Find the line labeled ‘PasswordAuthentication’. Remove the # from the beginning of the line, and change the yes to no. It should look like this when done: 

![Passwords Disabled](Images/ForwardAgentno.png)

This is a little tricky to test that it's working properly, I happen to have a spare standard user account on my host computer that I used to test. The best option would be to attempt to ssh in from another computer. You should be greeted with a message that the host has disallowed password access and the connection be terminated. 


## Install Anaconda Navigator
To use a Jupyter notebook on the server first the software has to be set up. If you are using a separate user account this can be done via the GUI. I chose not to use this route because the password to access that GUI was longer than I wanted to type without a password manager. Instead, it is also possible to install Anaconda entirely via the command line, just using the appropriate installer. 

On a client device start by going to the anaconda downloads page and finding the appropriate link. Python 3.7 for the Mac command line at the time of writing the link is 64-Bit Command-Line Installer (424 MB). Hang onto this link, you’ll need it in a moment. SSH into your host device using the instructions above. The best way to download this file is by using wget. Unfortunately, macOS doesn't come with wget built-in. In all likelihood you’re going to need it again in the future so if you would like you can go through the install steps to get wget, or you can use `curl -O ‘[link]’` where you replace [link] with the link to the download from earlier. 

Whichever process you use to get the package downloaded, use `ls` to check the name of the file, on mine, it downloaded as ‘Anaconda3-2019.10-MacOSX-x86_64.sh’ which is what I’ll use for the rest of this tutorial. Please replace this file name with the current version you are installing. 

In order to run the installer, you first have to adjust your permissions on the file. We do this using chmod: 

`chmod 711 Anaconda3-2019.10-MacOSX-x86_64.sh` 

The 711 in this sets the permissions such that you have full execution access to the file. From there you can execute it using the command 

`./Anaconda3-2019.10-MacOSX-x86_64.sh`

which will install the navigator in the current directory. At this point, you should have Anaconda installed. You can test this using ‘which jupyter’ which should show you the path of your current jupyter install. 

If you don't need the full functionality of Anaconda Navigator, it is possible to also install only JupyterLab using the command:

`pip install jupyterlab`

## Set Up and Test Connections
At this point, everything is configured and ready to go. From here its simply a matter of starting up the server and connecting to it. The Jupyter Server can now be launched by first SSHing into the host computer and using the command:

`Jupyter Lab --no-browser --port=9999` 

If you have already set something up on the host computer that uses port 9999 you can change that number to something else. For the duration of this tutorial, I’m going to use 9999 though. To connect it on your client device open up a new terminal window (or use cmd+t to open a new terminal tab) and enter the command: 

`ssh -p [your port] -N -f -L localhost:9000:localhost:9999 [remote username]@[remote ip]`

Where ‘your port’ is replaced with the port that you configured on your host computer to listen for SSH on, and the ‘remote username’ and ‘remote IP’ are the remote username and IP address you use in a normal ssh command. This creates an ‘SSH tunnel’ between the host computer and your client device. At this point, you can open up a browser and navigate to 

`localhost:9000/lab`

This port has been tunneled through and is working off of your host computer now, and this should launch the Jupyter lab in your browser. 

Congratulations! You now have a working remote Jupyter lab server. Any time that you want to use it just run the connection command, then SSH in and launch the server (if it isn’t already running). 

## Setting up Bash Shortcuts
Now, even though it's possible to run the server just as it is, it’s much easier to set up bash shortcuts so you don’t have to remember all the commands to make it work. Bash scripting is pretty easy, it’s mostly a matter of place the commands that you want to run in a file, allow execution access to the file, and place the file in the path. The first command is going to be ‘callHome’ (you can replace the name of the command with something more memorable for you if you would like). Start with the command: 

`nano callHome`

This both creates the text file and opens it up in nano. In this file enter the ssh command to connect to the host computer. It should look something like: 

`ssh -p [your port] remote@[your ip]`

Use control+x to exit, y to save, enter to keep the name. You now need to edit the permissions on the file to allow execution using the same chmod command from above:

`chmod 711 callHome`

And finally, move it into your path using the command:

`mv callHome /usr/local/bin/callhome`

This moves it into the ‘path’ allowing you to call this command with a simple ‘callhome’ in the terminal. You can test this by typing `which callhome` which will print out the path you just specified, or by typing in `callhome` and it should SSH into your host computer. 

Repeat the steps above with the file name ‘connectlab’ with the command:

`ssh -p [your port] -N -f -L localhost:9000:localhost:9999 remote@[remote ip]`

Then SSH into your host and repeat them for a file called ‘launchlab’ with the command:

`Jupyter Lab --no-browser --port=9999`

Once all of these shortcuts are in place it should be possible to launch your server by simply opening terminal on your computer and typing in these three commands: 

`connectlab
callhome
launchlab`

The connectlab and callhome are separate commands because you may want to ssh in multiple times to your host server but you only have to create the tunnel once. If you attempt to create the tunnel multiple times you will get an error message about the port already being in use. 
