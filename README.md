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
  Select pri/log 8.6 GB FREE SPACE --> Create a new partition --> Partition size 4.5 GB --> Primary --> Beginning --> Done setting up the partition

  Partition rest as you like.

12.  Configure the package manager: Use a network mirror? **<Yes>** -> Finland --> deb.debian.org --> *(Press enter when asked for proxy)*
	
13. Install GRUB?: **Yes** --> /dev/sda

14. Restart the VM and login with your root credentials.

15. type in following command to update the package list and upgrade your old packages
  ```
  apt update && apt upgrade -y
  ```
#### Partitioning revisited
	
When you are logged in, use command `cfdisk` to check thath your partition table looks something like this. 

![Disk partition](/assets/roger/partition_table.png)

if not, then you can resize it with cfdisk.



	
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
also edit sudoers file with command visudo

After `root ALL=(ALL:ALL) ALL`

Add a new line

`newuser ALL=(ALL:ALL) ALL`

Save the file and reboot!

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
	address 10.11.200.XXX
	netmask 255.255.255.252
	gateway 10.11.254.254
```
Remove /etc/interfaces.d/setup


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


- [x] You have to set the rules of your firewall on your server only with the services used
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

- [x] You have to set a DOS (Denial Of Service Attack) protection on your open ports of your VM.

	We need to install Fail2Ban with command `apt install fail2ban`
	
	First copy your jail file to local file. _(/etc/fail2ban/jail.conf is a file where you set-up rules for your DOS protection.)_
	
	`sudo cp /etc/fail2ban/jail.conf /etc/fail2ban/jail.local`
	Our current open ports are http 442 and ssh 5534
	
	**Example for ssh**
	Find ssh and add following lines:
	`enabled	= true`
	`port		= ssh, 5534`
	
	![instructins here](https://blog.rapid7.com/2017/02/13/how-to-protect-ssh-and-apache-using-fail2ban-on-ubuntu-linux/)

	
- [x] You have to set a protection against scans on your VM’s open ports.

To protect yourself against port scanning, we need to install software called Portsentry

Install it with command: sudo apt install portsentry

We need to modify some config files _(/etc/default/portsentry , /etc/portsentry/portsentry.conf)_ before out protection works.

In file /etc/default/portsentry Change 

TCP_MODE="tcp"
UPD_MODE="upd"

to

TCP_MODE="atcp"
UPD_MODE="aupd"

In file /etc/portsentry/portsentry.conf change lines
_line 135_
BLOCK_UDP="0"
_line 136_
BLOCK_TCP="0"

_line 209_

Uncomment line 209

KILL_ROUTE="/sbin/iptables -I INPUT -s $TARGET$ -j DROP"
After the changes, restart the service: `sudo service portsentry restart`

- [x] Stop the services you don’t need for this project.

You can see your current processes with command 

`sudo service --status-all`

There you can see all the services that are currently running. You can stop the serices with following command:

`sudo systemctl stop SERVICENAME`

Example: `sudo systemctl stop dbus`

And to disable them frm starting at the boot. You can use **disable** option

Example:

`sudo systemctl disable dbus`


- [] Create a script that updates all the sources of package, then your packages and
which logs the whole in a file named /var/log/update_script.log. Create a scheduled
task for this script once a week at 4AM and every time the machine reboots.


### 1. Create script

_update_script_

```console
#!/bin/bash
LOG_FILE=/var/log/update_script.log
if [ -f "$LOG_FILE" ]; then
	sudo chmod 666 $LOG_FILE
else
	sudo touch $LOG_FILE
	sudo chmod 666 $LOG_FILE
fi
sudo apt update -y && sudo apt upgrade -y 1> $LOG_FILE
echo "System script done" | mail -s "System update script" regular
```
### 2. Create scheduled task ** Versin 1 **
You should create cronjob?!

* Once a week 4 AM
 
 Move file to cron weekly folder (/etc/cron.weekly) 
 remove any extensions like .sh at the end of the file. Otherwise it won't run.
 

```console
# /etc/crontab: system-wide crontab
# Unlike any other crontab you don't have to run the `crontab'
# command to install the new version when you edit this file
# and files in /etc/cron.d. These files also have username fields,
# that none of the other crontabs do.

SHELL=/bin/sh
PATH=/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin

# Example of job definition:
# .---------------- minute (0 - 59)
# |  .------------- hour (0 - 23)
# |  |  .---------- day of month (1 - 31)
# |  |  |  .------- month (1 - 12) OR jan,feb,mar,apr ...
# |  |  |  |  .---- day of week (0 - 6) (Sunday=0 or 7) OR sun,mon,tue,wed,thu,fri,sat
# |  |  |  |  |
# *  *  *  *  * user-name command to be executed
17 *	* * *	root    cd / && run-parts --report /etc/cron.hourly
25 6	* * *	root	test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.daily )
0  4	* * 7	root	test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.weekly )
52 6	1 * *	root	test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.monthly )
#
```

To test that this works. Use command: `sudo run-parts -v /etc/cron.weekly`


#### Everytime machine reboots
 
`mv update_script.sh /etc/cron.weekly/update_script`

Now edit your crontab file with command `sudo crontab -e` and add line `@reboot /etc/cron.weekly/update_script`

Now reboot!


- [] Make a script to monitor changes of the /etc/crontab file and sends an email to
root if it has been modified. Create a scheduled script task every day at midnight.

##### Script

```console
#!/bin/bash
if [ -z $(sudo find /etc/crontab -mmin -1440) ]
then
	exit;
else
	echo "Crontab has been changed" | mail -s "Crontab notification" root
fi
```
Explanation:

	-z $(sudo find /etc/crontab --mmin -1440)
	
	-z means that it should return 0
	
	-mmin flag at find means that it will return filename if the file has been modifien in 1440 minutes

For security reasons it's good to redirect your rot mail to another user that has less rights. To do this modify /etc/aliases file.

`sudo vim /etc/aliases`

add following line

`root: regular`
and save. (regular == users name)

#### Version 2


Create a folder /root/scripts and copy scripts in there

Modify crontab with command: `sudo crontab -e`

```console

# Edit this file to introduce tasks to be run by cron.
#
# Each task to run has to be defined through a single line
# indicating with different fields when the task will be run
# and what command to run for the task
#
# To define the time you can provide concrete values for
# minute (m), hour (h), day of month (dom), month (mon),
# and day of week (dow) or use '*' in these fields (for 'any').
#
# Notice that tasks will be started based on the cron's system
# daemon's notion of time and timezones.
#
# Output of the crontab jobs (including errors) is sent through
# email to the user the crontab file belongs to (unless redirected).
#
# For example, you can run a backup of all your user accounts
# at 5 a.m every week with:
# 0 5 * * 1 tar -zcf /var/backups/home.tgz /home/
#
# For more information see the manual pages of crontab(5) and cron(8)
#
# m h  dom mon dow   command
@reboot /root/scripts/update_script
0 4 * * * /root/scripts/update_script
0 0 * * * /root/scripts/cronwatcher

```

Save and reboot to check that it works


## SSL

https://wiki.debian.org/Self-Signed_Certificate
