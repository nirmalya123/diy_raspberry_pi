# Create a NAS using Raspberry Pi and a USB-HDD
The document contains steps to create a network attached storage using a usb-hdd and a Raspberry Pi(or **Rpi**) attached in a private wifi network.
Some basic security elements are added. References mentioned can be used for betterment.

### 1. Hardware connection and usage-
- Here **RASPBERRY PI 2 MODEL B** was used with a usb-wifi-adapter. Using higher versions of Rpi with built-in wifi will make life easier. When I first made this in 2016 I had to manually install the drivers for the wifi adapter. Raspbian images from late 2017 has the relevant drivers installed already. :relaxed:
- Here [Raspbian Stretch With Desktop](https://www.raspberrypi.org/downloads/raspbian/) is used. [Installation instructions.](https://www.raspberrypi.org/documentation/installation/installing-images/README.md). If you are planning to go for a headless version(i.e. without gui) you may choose *Raspbian Stretch LITE*.
- The spinning 500GB HDD is attached to one of the usb ports of Rpi. To allow more power/current for Rpi versions below RPi-3 append the below line in ```/boot/config.txt```
This would increase maximum usb current from 600mA to 1200 mA.

	`max_usb_current=1`

### 2. Enhancing basic security
By default Raspbian comes with default user ```pi``` with password ```raspberry```. It better to remove this user and add a new user. [Reference](https://www.raspberrypi.org/documentation/configuration/security.md).
1. Create a new user- `sudo adduser <user1_name>`
1. Add the new used in the sudoer list-`sudo adduser <user name> sudo`
1. Ask for password for sudo-
	- Open the file `sudo nano /etc/sudoers.d/010_pi-nopasswd`
	- Modify the line as - `pi ALL=(ALL) PASSWD: ALL`
	- Save end exit with `CTRL+X`
1. Remove the default `pi` user- `sudo deluser -remove-home pi`

### 3. Installing required packages
I generally prefer to login to shell used sudo so that I do not need to type sudo every time.
This can be done using `sudo /bin/bash`.
Alternatively you can use the general way.
1.	#### Update and upgrade the current installation to the latest version-
	```
	sudo apt-get update
	sudo apt-get upgrade
	```
1.  #### Enable ssh login
Remote login useing ssh can be enabled either from `raspi-config` or installing `openssh-server` using `sudo apt-get install openssh-server`.
1.  #### Install packages for sharing files and folders
`sudo apt-get -yf install samba samba-common-bin`
1.  #### Install package for mounting file system of usb-hdd. In my case it is ntfs.
`sudo apt-get -yf install ntfs-3g`

	> The `-yf` option is for --assume-yes and --fix-broken. [Ref.](https://linux.die.net/man/8/apt-get0)

### 4. Mount HDD
1. #### Check the disk config using
	Run `sudo blkid`. Output will be something like below-
```
/dev/sda1: LABEL="New Volume" UUID="50E******CEDED**" TYPE="ntfs" PARTUUID="d9****84-01"
/dev/sda2: LABEL="New Volume" UUID="D*****CC5CDC***3" TYPE="ntfs" PARTUUID="d9****84-02"
```
	Note the `UUID`. In some cases I found usage of `PARTUUID`
1. #### Create folder for mounting
	`mkdir -p /media/HDD1`

	The `-p` option created all the parent directories if they do not exist. Generally mount points are in `/mnt` or `/media`.
1. #### Mount the hdd
	`sudo mount -t auto /dev/sda2 /media/HDD1`

	Check if mount is successful using `ls /media/HDD1`. For success the content of the disk will be displayed.

	To automatically mount the disk each time the system is rebooted modify `/etc/fstab` file. Append the below line in the file.
	> **It is always advisable to take backup of the config files before doing any alterations.** `nano -Bw <file name>` automatically creates a backup of file being modified.

	```
	UUID=D*****CC5CDC***3 /media/HDD1 ntfs-3g defaults,nofail,auto,umask=000,users,rw 0 0
	```
	> Please note the UUID should be as per the output of `blkid` in your system.

	You may use `uid` and `gid` from [`id`](https://kb.iu.edu/d/adwf) command.

### 4. Configure samba
1. Open `smb.conf` using `sudo nono -Bw /etc/samba/smb.conf`
1. Append as per the below block-
	```
	[RaspberryPi NAS]
		comment = NAS
		public = no
		writeable = yes
		browsable = yes
		path = /media/HDD1
		create mask = 0700
		directory mask = 0700
	```
	> `public = no` will ask for username and password whenever connection would be made. `public = yes` will directly give access to the content without any authentication.
1. Save and exit the file using `CTRL+X`.
1. Add the user in the samba database- `sudo smbpasswd - a <user1_name>`.
1. Restart the samba service- `sudo service smbd restart`. If that does not work you may try `sudo /etc/init.d/samba restart`.

### 5. Connect from Windows
From **This PC** got to **Map network drive**. In the option under the **Computer** tab you can either type in the share location with IP or can browse to that network location. Enter the username and password when asked. The location would be added as a new drive in the computer.

>*My conclusion* - It was fun then and now to create a NAS of my own. But cooling is not good in such devices even though some heatsink is attached. Also security may sometimes be a problem. Rpi has usb2.0 so speed is another problem. Best option is to invest in some good NAS from Western digital or Seagate.
