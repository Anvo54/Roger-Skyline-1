# Roger skyline 1

*This document is an initiation to system and network administration*

## Virtual machine part 

## Required goals

- [x] Install Linux distro with Virtualbox
- [x] Disk size must be 8 GB
- [x] Have at least one 4,2 GB partition
- [x] Update the packages

### Installing the linux distro

Requirements:

* Linux ISO image
* Virtualbox

At this example I am using [**Debian linux version 10.2.0**](https://cdimage.debian.org/debian-cd/current-live/amd64/iso-hybrid/debian-live-10.2.0-amd64-standard.iso)

*This tutorial is assuming you to have Virtualbox already installed.*

At Virtualbox press **New** button at the let corner.

![New virtual machine](/assets/roger/1.png)

1. Give your virtual machine name. In this case I typed in debian and type changed automatially to Linux and distro changed to Debian (64 Bit). **Press continue**

![Add name to new virtual machine](/assets/roger/2.png)
***

2. Recommended memory size 1024MB is fine for our purpose. **Press continue**

3. Choose **Create virtual hard disk now** and click Continue

4. Choose VDI and click continue

![Create a disk image](/assets/roger/3.png)
***
5. Let's choose fixed size for this example and click continue

6. Give name for the diskimage and set size at 8 GB. Choose a folder on your computer where you want to save it. (*At te schools mac's use __goinfre__ folder*) Click **Create**

7. Right click on your new virtualmachine and press **Settings**. Select storage --> Empty CD image --> Choose Virtual Optical Disk file. Select your freshly downloaded ISO file.

![Virtualbox settings](/assets/roger/4.png)

***

![Virtualbox settings](/assets/roger/5.png)

***

8. Then go to network settings and select following settings **Attached to: Bridged Adapter & Name en0: Ethernnet**. Click OK
9. **Click Start**

10. Choose Debian installer and follow the instructions on the screen. *Below you can see my steps.*
  
  ***
  
![Debian install](/assets/roger/6.png)
  
  ***
  
  * Language : English
  * Select your location: Other -> Europe -> Finland
  * Configure locales: United States
  * Configure keyboard: American English
  * Configure the network / hostname: debian
  * Configure the network / domanin name:
  * Set up users and passwords: Give your root user a password that you will remember
  * Give your regullar user a Full name (Firstname Lastname): regular user
  * Username for the same user (Usually firstname of the user): regular
  * Password for the new user. Something you remember
 
11. Partitioning the disk

***

![Partition disk](/assets/roger/7.png)
  
***

  For partition method, select **Manual**.
  Select your VBOX HARDDISK. 
  Create new empty partition table on this device? select **Yes**
  Select pri/log 8.6 GB FREE SPACE --> Create a new partition --> Partition size 4.2 GB --> Primary --> Beginning --> Done setting up the partition
  
***
  
  ![Partition disk](/assets/roger/8.png)
  
***

  ![Partition disk](/assets/roger/9.png)

***

Select pri/log 4.4 GB FREE SPACE --> Create a new partition --> Partition size 2.4 GB --> Logical --> Beginning --> (Mount point: /home) Done setting up the partition
  
  Select pri/log 2.0 GB FREE SPACE --> Create a new partition --> Partition size 2.0 GB --> Logical --> Beginning --> Use as: swap area --> Done setting up the partition --> Finish partitioning and write changes to disk & Yes

12.  Configure the package manager: Use a network mirror? **<Yes>** -> Finland --> deb.debian.org --> *(Press enter when asked for proxy)*
13. Install GRUB?: **Yes** --> /dev/sda
14. Restart the VM and login with your root credentials.
15. type in following command to update the package list and upgrade your old packages
  ```
  apt update && apt upgrade -y
  ```
#### Partitioning revisited
	
When you are logged in, use command `cfdisk` to resize some of your freshly created partition to 4.2G.

RESULT BELOW

***

![Disk partition](/assets/roger/partition.png)

***
	
## Network and Security Part
 
 - [x] You must create a non-root user to connect to the machine and work.


To add a new user, use command: adduser username
then follow the instructions.

**Example**

```
	adduser fourtytwo
```

- [x] Use sudo, with this user, to be able to perform operation requiring special rights.
To add regular user to sudoers group use usermod command with flags aG

a = add user to group
G = group to be added

followed by group name and username

**Example**
```
usermod -aG sudo fourtytwo
```
- [x] We don’t want you to use the DHCP service of your machine. You’ve got to
configure it to have a static IP and a Netmask in \30.

1. Edit the /etc/network/interfaces

and change the content to 

```
# This file describes the network interfaces available on your system
# and how to activate them. For more information, see interfaces(5).

source /etc/network/interfaces.d/*

# The loopback network interface
auto lo
iface lo inet loopback

# The primary network interface
auto enp0s3
```
and create a new file */etc/network/interfaces.d/enp0s3*

```
iface enp0s3 inet static
	address 10.11.200.54
	netmask 255.255.255.252
	gateway 10.11.254.254
```

Now reboot your VM with command `sudo reboot`

- [x] _You have to change the default port of the SSH service by the one of your choice.
SSH access HAS TO be done with publickeys. SSH root access SHOULD NOT
be allowed directly, but with a user who can be root._

First you need to make a keypair from your mac.

Use command `ssh-keygen`

Enter filename you want and password.

Now you should find the newly created file in your current folder.

Nexy you should copy the keyfile to VM with followiing command: `ssh-copy-id -i FILENAME.pub USERNAME@IPADDRESS -p PORT`

	`ssh-copy-id -i vm.pub regular@10.11.200.54 -p 5534`

To connect your server using ssh key. You can use -i flag to specify the keyfile.

	`ssh -i FILENAME -p 'PORT' 'USER@IPADDRESS'`

Example:
	`ssh -i vm -p '5534' 'regular@10.11.200.54'`

Next to change the default ssh port, you need to modify `/etc/ssh/sshd_config` file on your VM.

Find the lines `#Port 22`, `#permitRootLogin prohibit-password`, `#PasswordAuthentication yes`
Remove the comment sign from all of them (#) and change to port number to number you want. Example Port 5534

permitRootLogin no
PasswordAuthentication no

Save the file and restart your ssh with command `service ssh restart`


_try to connect to your VM again_ `ssh -i vm -p '5534' 'regular@10.11.200.54'`


- [] You have to set the rules of your firewall on your server only with the services used
outside the VM.

First you need to install ufw (Uncomplicated Firewall) with command `sudo apt install ufw`

![Basic UFW commands](https://wiki.ubuntu.com/UncomplicatedFirewall#Basic_Usage)

First we need to enable our firewall `sudo ufw enable`.

Good practice is to disable all connections from outside world and open only the necessary

`ufw default deny incoming`
`ufw default allow outgoing`

Then we need to allow ports that we use.

Ports:
5534 is our new ssh port
443 is secure https port

Example:

`sudo ufw 443`

- [] You have to set a DOS (Denial Of Service Attack) protection on your open ports of your VM.

	We need to install Fail2Ban with command `apt install fail2ban`
	
	First copy your jail file to local file. _(/etc/jail.conf is a file where you set-up rules for your DOS protection.)_
	
	`sudo cp /etc/jail.conf /etc/jail.local`
	Our current open ports are http 442 and ssh 5534
	
	**Example for ssh**
	Find ssh and add following lines:
	`enabled	= true`
	`port		= ssh, 5534`
	
	Rest below.
```
#
# HTTP servers
#

[apache-auth]

port     = http,https
logpath  = %(apache_error_log)s
enabled	 = true

[apache-badbots]
# Ban hosts which agent identifies spammer robots crawling the web
# for email addresses. The mail outputs are buffered.
port     = http,https
logpath  = %(apache_access_log)s
bantime  = 48h
maxretry = 1
enabled	= true

[apache-noscript]

port     = http,https
logpath  = %(apache_error_log)s
enabled = true

[apache-overflows]

port     = http,https
logpath  = %(apache_error_log)s
maxretry = 2
enabled = true

[apache-nohome]

port     = http,https
logpath  = %(apache_error_log)s
maxretry = 2
enabled = true

[apache-botsearch]

port     = http,https
logpath  = %(apache_error_log)s
maxretry = 2
enabled = true

[apache-fakegooglebot]

port     = http,https
logpath  = %(apache_access_log)s
maxretry = 1
ignorecommand = %(ignorecommands_dir)s/apache-fakegooglebot <ip>
enabled = true

[apache-modsecurity]

port     = http,https
logpath  = %(apache_error_log)s
maxretry = 2
enabled = true

[apache-shellshock]

port    = http,https
logpath = %(apache_error_log)s
maxretry = 1
enabled = true

[openhab-auth]

filter = openhab
action = iptables-allports[name=NoAuthFailures]
logpath = /opt/openhab/logs/request.log
```
	
	
- [] You have to set a protection against scans on your VM’s open ports.

To protect yourself against port scanning, we need to install software called Portsentry

https://blog.rapid7.com/2017/06/24/how-to-install-and-use-psad-ids-on-ubuntu-linux/

- [] Stop the services you don’t need for this project.



- [] Create a script that updates all the sources of package, then your packages and
which logs the whole in a file named /var/log/update_script.log. Create a scheduled
task for this script once a week at 4AM and every time the machine reboots.



- [] Make a script to monitor changes of the /etc/crontab file and sends an email to
root if it has been modified. Create a scheduled script task every day at midnight.
