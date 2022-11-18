\*\* Setup Raspberry Pi as Second MCU

SET THE RASPBERRY PI AS A SECONDARY MCU
https://www.klipper3d.org/RPi_microcontroller.html

This gives you control over the GPIOs. Now you can make macros to runs LEDs off and on, fans, read sensors etc…

INSTALL THE RC SCRIPT
After installing Klipper, install the script. run:

cd ~/klipper/
sudo cp "./scripts/klipper-mcu-start.sh" /etc/init.d/klipper_mcu
sudo update-rc.d klipper_mcu defaults
cd ~/klipper/
sudo cp "./scripts/klipper-mcu-start.sh" /etc/init.d/klipper_mcu
sudo update-rc.d klipper_mcu defaults

BUILDING THE MICRO-CONTROLLER CODE
To compile the Klipper micro-controller code for a secondary mcu we start by configuring it for the “Linux process”, re-run the commands:

cd ~/klipper/
make menuconfig
To build and install the new micro-controller code, run:
sudo service klipper stop
make flash
sudo service klipper start
If you get a permission denied error: '/tmp/klipper_host_mcu' you can add yourself to the tty group.
sudo usermod -a -G tty pi
Then reboot. If all else fails and it still produces a permission denied error you can (not recommended): sudo chmod 777 /home/pi/virtual_sdcard.

\*\* Setup Users

from https://www.digitalocean.com/community/tutorials/how-to-add-and-delete-users-on-ubuntu-20-04

sudo usermod -aG pi,adm,tty,dialout,cdrom,audio,video,plugdev,games,users,input,netdev,spi,i2c,gpio hos

Introduction
Adding and removing users on a Linux system is one of the most important system administration tasks to familiarize yourself with. When you create a new system, you are often only given access to the root account by default.

While running as the root user gives you complete control over a system and its users, it is also dangerous and possibly destructive. For common system administration tasks, it’s a better idea to add an unprivileged user and carry out those tasks without root privileges. You can also create additional unprivileged accounts for any other users you may have on your system. Each user on a system should have their own separate account.

For tasks that require administrator privileges, there is a tool installed on Ubuntu systems called sudo. Briefly, sudo allows you to run a command as another user, including users with administrative privileges. In this guide, you’ll learn how to create user accounts, assign sudo privileges, and delete users.

Prerequisites
To complete this tutorial, you will need access to a server running Ubuntu 20.04. Ensure that you have root access to the server and firewall enabled. To set this up, follow our Initial Server Setup Guide for Ubuntu 20.04.

Adding a User
If you are signed in as the root user, you can create a new user at any time by running the following:

adduser newuser
If you are signed in as a non-root user who has been given sudo privileges, you can add a new user with the following command:

sudo adduser newuser
Either way, you will be required to respond to a series of questions:

Assign and confirm a password for the new user.
Enter any additional information about the new user. This is optional and can be skipped by pressing ENTER if you don’t wish to utilize these fields.
Finally, you’ll be asked to confirm that the information you provided was correct. Press Y to continue.
Your new user is now ready for use and can be logged into with the password that you entered.

If you need your new user to have administrative privileges, continue on to the next section.

Granting a User Sudo Privileges
If your new user should have the ability to execute commands with root (administrative) privileges, you will need to give the new user access to sudo. Let’s examine two approaches to this task: first, adding the user to a pre-defined sudo user group, and second, specifying privileges on a per-user basis in sudo’s configuration.

Adding the New User to the Sudo Group
By default, sudo on Ubuntu 20.04 systems is configured to extend full privileges to any user in the sudo group.

You can view what groups your new user is in with the groups command:

groups newuser
Output
newuser : newuser
By default, a new user is only in their own group because adduser creates this in addition to the user profile. A user and its own group share the same name. In order to add the user to a new group, you can use the usermod command:

usermod -aG sudo newuser
The -aG option tells usermod to add the user to the listed groups.

Please note that the usermod command itself requires sudo privileges. This means that you can only add users to the sudo group if you’re signed in as the root user or as another user that has already been added as a member of the sudo group. In the latter case, you will have to precede this command with sudo, as in this example:

sudo usermod -aG sudo newuser
Specifying Explicit User Privileges in /etc/sudoers
As an alternative to putting your user in the sudo group, you can use the visudo command, which opens a configuration file called /etc/sudoers in the system’s default editor, and explicitly specify privileges on a per-user basis.

Using visudo is the only recommended way to make changes to /etc/sudoers because it locks the file against multiple simultaneous edits and performs a validation check on its contents before overwriting the file. This helps to prevent a situation where you misconfigure sudo and cannot fix the problem because you have lost sudo privileges.

If you are currently signed in as root, run the following:

visudo
If you are signed in as a non-root user with sudo privileges, run the same command with the sudo prefix:

sudo visudo
Traditionally, visudo opened /etc/sudoers in the vi editor, which can be confusing for inexperienced users. By default on new Ubuntu installations, visudo will use the nano text editor, which provides a more convenient and accessible text editing experience. Use the arrow keys to move the cursor, and search for the line that reads like the following:

/etc/sudoers
root ALL=(ALL:ALL) ALL
Below this line, add the following highlighted line. Be sure to change newuser to the name of the user profile that you would like to grant sudo privileges:

/etc/sudoers
root ALL=(ALL:ALL) ALL
newuser ALL=(ALL:ALL) ALL
Add a new line like this for each user that should be given full sudo privileges. When you’re finished, save and close the file by pressing CTRL + X, followed by Y, and then ENTER to confirm.

# **SAMBA Setup - GCODE File Network Share Setup**

To set up a network file share, making your gcode and Klipper config files available to all your network-attached devices - either windows or mac file explorers, perform the following once ssh'd into your Klipper rPi. Note that this samba configuration is 'open' and therefore accessible to any device attached to the same LAN, just as the Mainsail HTTP interface is. While it is possible to restrict access to these samba file shares, I feel it is a moot point given the nature of the rest of the software suite, so I will not cover that here. If this is worrisome to you, please research alternate configurations.

-Install file serving smb service:  
$`sudo apt-get install samba winbind -y`

-Configure file share. Add the following to the end of the smb.conf file:  
$`sudo nano /etc/samba/smb.conf`

```
[orebot]
   comment = Orebot GCode Files
   path = /home/pi/gcode_files
   browseable = Yes
   writeable = Yes
   only guest = no
   create mask = 0777
   directory mask = 0777
   public = yes
   read only = no
   force user = root
   force group = root

[orebot-klipper_config]
   comment = Orebot Klipper Configurations
   path = /home/pi/klipper_config
   browseable = Yes
   writeable = Yes
   only guest = no
   create mask = 0777
   directory mask = 0777
   public = yes
   read only = no
   force user = root
   force group = root
```

\*\* Enable Samba User

root@orebot:/home/pi/klipper# sudo smbpasswd -a hos
New SMB password:
Retype new SMB password:
Added user hos.
root@orebot:/home/pi/klipper# sudo smbpasswd -a pi
New SMB password:
Retype new SMB password:
Added user pi.
root@orebot:/home/pi/klipper# sudo smbpasswd -e pi
Enabled user pi.
root@orebot:/home/pi/klipper# sudo smbpasswd -e hos
Enabled user hos.

#### Install Klipper Z Calibration

https://github.com/protoloft/klipper_z_calibration#how-to-install-it

## How To Install It

Clone this repo and run the `install.sh` script. Like:

```bash
cd /home/pi
git clone https://github.com/protoloft/klipper_z_calibration.git
./klipper_z_calibration/install.sh
```

It's safe to execute the install script multiple times.

More on this in the [Moonraker Update Manager](#moonraker-update-manager) section.

### Moonraker Update Manager

> :bulb: **NEW:** With a current Moonraker version, the dummy service is not
> necessary anymore. If you have updated to the 0.9.1 version, you need to adapt the
> following configuration block in your "moonraker.conf" file. The old dummy service
> can be removed by executing the install script again. Like:
> `/home/pi/klipper_z_calibration/install.sh`

It's possible to keep this extension up to date with the Moonraker's update manager by
adding this configuration block to the "moonraker.conf" of your printer:

```text
[update_manager client z_calibration]
type: git_repo
path: ~/klipper_z_calibration
origin: https://github.com/protoloft/klipper_z_calibration.git
install_script: install.sh
managed_services: klipper
```

This requires this repository to be cloned into your home directory (e.g. /home/pi):

```bash
git clone https://github.com/protoloft/klipper_z_calibration.git
```

The install script assumes that Klipper is also installed in your home directory under
"klipper": `${HOME}/klipper`.

> :point_up: **NOTE:** If your Moonraker is not on a recent version, you may get an error
> with the "managed_services" line!
