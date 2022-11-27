## Bigtreetech Octopus Pro

## Setup Raspberry Pi as Second MCU

See information here https://www.klipper3d.org/RPi_microcontroller.html

This gives you control over the GPIOs. Now you can make macros to runs LEDs off and on, fans, read sensors etc…

### INSTALL THE RC SCRIPT

After installing Klipper, install the script. run:

```sh
cd ~/klipper/
sudo cp "./scripts/klipper-mcu-start.sh" /etc/init.d/klipper_mcu
sudo update-rc.d klipper_mcu defaults
```

### BUILDING THE MICRO-CONTROLLER CODE

To compile the Klipper micro-controller code for a secondary mcu we start by configuring it for the “Linux process”, re-run the commands:

```sh
cd ~/klipper/
make menuconfig
To build and install the new micro-controller code, run:
sudo service klipper stop
make flash
sudo service klipper start
```

If you get a permission denied error: `'/tmp/klipper_host_mcu'` you can add yourself to the tty group.

```sh
sudo usermod -a -G tty pi
```

Then reboot. If all else fails and it still produces a permission denied error you can (not recommended): `sudo chmod 777 /home/pi/virtual_sdcard`.

## Setup Users

See details here https://www.digitalocean.com/community/tutorials/how-to-add-and-delete-users-on-ubuntu-20-04

`sudo usermod -aG pi,adm,tty,dialout,cdrom,audio,video,plugdev,games,users,input,netdev,spi,i2c,gpio hos`

### Introduction

Adding and removing users on a Linux system is one of the most important system administration tasks to familiarize yourself with. When you create a new system, you are often only given access to the root account by default.

While running as the root user gives you complete control over a system and its users, it is also dangerous and possibly destructive. For common system administration tasks, it’s a better idea to add an unprivileged user and carry out those tasks without root privileges. You can also create additional unprivileged accounts for any other users you may have on your system. Each user on a system should have their own separate account.

For tasks that require administrator privileges, there is a tool installed on Ubuntu systems called sudo. Briefly, sudo allows you to run a command as another user, including users with administrative privileges. In this guide, you’ll learn how to create user accounts, assign sudo privileges, and delete users.

### Prerequisites

To complete this tutorial, you will need access to a server running Ubuntu 20.04. Ensure that you have root access to the server and firewall enabled. To set this up, follow our Initial Server Setup Guide for Ubuntu 20.04.

### Adding a User

If you are signed in as the root user, you can create a new user at any time by running the following:

adduser newuser
If you are signed in as a non-root user who has been given sudo privileges, you can add a new user with the following command:

sudo adduser newuser
Either way, you will be required to respond to a series of questions:

### Assign and confirm a password for the new user.

Enter any additional information about the new user. This is optional and can be skipped by pressing ENTER if you don’t wish to utilize these fields.
Finally, you’ll be asked to confirm that the information you provided was correct. Press Y to continue.
Your new user is now ready for use and can be logged into with the password that you entered.

If you need your new user to have administrative privileges, continue on to the next section.

### Granting a User Sudo Privileges

If your new user should have the ability to execute commands with root (administrative) privileges, you will need to give the new user access to sudo. Let’s examine two approaches to this task: first, adding the user to a pre-defined sudo user group, and second, specifying privileges on a per-user basis in sudo’s configuration.

### Adding the New User to the Sudo Group

By default, sudo on Ubuntu 20.04 systems is configured to extend full privileges to any user in the sudo group.

You can view what groups your new user is in with the groups command:

groups newuser
Output
newuser : newuser

By default, a new user is only in their own group because adduser creates this in addition to the user profile. A user and its own group share the same name. In order to add the user to a new group, you can use the usermod command:

```sh
usermod -aG sudo newuser
```

The `-aG` option tells usermod to add the user to the listed groups.

Please note that the usermod command itself requires sudo privileges. This means that you can only add users to the sudo group if you’re signed in as the root user or as another user that has already been added as a member of the sudo group. In the latter case, you will have to precede this command with sudo, as in this example:

```sh
sudo usermod -aG sudo newuser
```

### Specifying Explicit User Privileges in /etc/sudoers

As an alternative to putting your user in the sudo group, you can use the visudo command, which opens a configuration file called /etc/sudoers in the system’s default editor, and explicitly specify privileges on a per-user basis.

Using visudo is the only recommended way to make changes to /etc/sudoers because it locks the file against multiple simultaneous edits and performs a validation check on its contents before overwriting the file. This helps to prevent a situation where you misconfigure sudo and cannot fix the problem because you have lost sudo privileges.

If you are currently signed in as root, run the following:

```sh
visudo
```

If you are signed in as a non-root user with sudo privileges, run the same command with the sudo prefix:

sudo visudo
Traditionally, visudo opened /etc/sudoers in the vi editor, which can be confusing for inexperienced users. By default on new Ubuntu installations, visudo will use the nano text editor, which provides a more convenient and accessible text editing experience. Use the arrow keys to move the cursor, and search for the line that reads like the following:

```sh
/etc/sudoers
root ALL=(ALL:ALL) ALL
```

Below this line, add the following highlighted line. Be sure to change newuser to the name of the user profile that you would like to grant sudo privileges:

```sh
/etc/sudoers
root ALL=(ALL:ALL) ALL
newuser ALL=(ALL:ALL) ALL
```

Add a new line like this for each user that should be given full sudo privileges. When you’re finished, save and close the file by pressing CTRL + X, followed by Y, and then ENTER to confirm.

## SAMBA Setup - GCODE File Network Share Setup\*\*

To set up a network file share, making your gcode and Klipper config files available to all your network-attached devices - either windows or mac file explorers, perform the following once ssh'd into your Klipper rPi. Note that this samba configuration is 'open' and therefore accessible to any device attached to the same LAN, just as the Mainsail HTTP interface is. While it is possible to restrict access to these samba file shares, I feel it is a moot point given the nature of the rest of the software suite, so I will not cover that here. If this is worrisome to you, please research alternate configurations.

###Install file serving smb service:  
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

### Enable Samba User

```sh
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
```

## Install Klipper Z Calibration

See details here https://github.com/protoloft/klipper_z_calibration#how-to-install-it

### How To Install It

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

## Setup WLED for Moonraker

### [wled]

Enables control of a WLED strip. Moonraker always supports 4 color channel strips - the color order is defined within WLED itself.

#### moonraker.conf

```cfg
[wled strip_name]
type:
#   The type of device. Can be either http, or serial.
#   This parameter must be provided.
address:
#   The address should be a valid ip or hostname for the wled webserver.
#   Required when type: http
serial:
#   The serial port to be used to communicate directly to wled. Requires wled
#   0.13 Build 2108250 or later.
#   Required when type: serial
initial_preset:
#   Initial preset ID (favourite) to use. If not specified initial_colors
#   will be used instead.
initial_red:
initial_green:
initial_blue:
initial_white:
#   Initial colors to use for all neopixels should initial_preset not be set,
#   initial_white will only be used for RGBW wled strips (defaults: 0.5)
chain_count:
#   Number of addressable neopixels for use (default: 1)
color_order:
#   *** DEPRECATED - Color order is defined per GPIO in WLED directly ***
#   Color order for WLED strip, RGB or RGBW (default: RGB)
```

Below are some examples:

#### moonraker.conf

```cfg
[wled case]
type: http
address: led1.lan
initial_preset: 45
chain_count: 76

[wled lounge]
type: http
address: 192.168.0.45
initial_red: 0.5
initial_green: 0.4
initial_blue: 0.3
chain_count: 42

[wled stealthburner]
type: serial
serial: /dev/serial/by-id/usb-1a86_USB_Serial-if00-port0
initial_white: 0.6
chain_count: 3
It is possible to control wled from the klippy host, this can be done using one or more macros, such as:
```

#### printer.cfg

```cfg
[gcode_macro WLED_ON]
description: Turn WLED strip on using optional preset and resets led colors
gcode:
  {% set strip = params.STRIP|string %}
  {% set preset = params.PRESET|default(-1)|int %}

  {action_call_remote_method("set_wled_state",
                             strip=strip,
                             state=True,
                             preset=preset)}

[gcode_macro WLED_CONTROL]
description: Control effect values and brightness
gcode:
  {% set strip = params.STRIP|default('lights')|string %}
  {% set brightness = params.BRIGHTNESS|default(-1)|int %}
  {% set intensity = params.INTENSITY|default(-1)|int %}
  {% set speed = params.SPEED|default(-1)|int %}

  {action_call_remote_method("set_wled_state",
                             strip=strip,
                             brightness=brightness,
                             intensity=intensity,
                             speed=speed)}

[gcode_macro WLED_OFF]
description: Turn WLED strip off
gcode:
  {% set strip = params.STRIP|string %}

  {action_call_remote_method("set_wled_state",
                             strip=strip,
                             state=False)}

[gcode_macro SET_WLED]
description: SET_LED like functionality for WLED, applies to all active segments
gcode:
    {% set strip = params.STRIP|string %}
    {% set red = params.RED|default(0)|float %}
    {% set green = params.GREEN|default(0)|float %}
    {% set blue = params.BLUE|default(0)|float %}
    {% set white = params.WHITE|default(0)|float %}
    {% set index = params.INDEX|default(-1)|int %}
    {% set transmit = params.TRANSMIT|default(1)|int %}

    {action_call_remote_method("set_wled",
                               strip=strip,
                               red=red, green=green, blue=blue, white=white,
                               index=index, transmit=transmit)}
```

## Bed mesh after every 10th print

Details found here: https://www.reddit.com/r/klippers/comments/qdr0g9/my_gcode_macro_to_mesh_level_the_bed_every_10th/

```sh
[save_variables]
filename: ~/klipper_config/variables.cfg ; variable storage file

[gcode_macro START_PRINT]
gcode:
    {% set BED_TEMP = params.BED_TEMP|default(60)|float %}
    {% set EXTRUDER_TEMP = params.EXTRUDER_TEMP|default(190)|float %}
    M104 S{EXTRUDER_TEMP} ; start heating extruder
    M190 S{BED_TEMP} ; wait for bed temperature
    G28 ; Home
    LEVEL_BED_ADVANCED MAX_AGE=10 ; probe mesh eventually
    ... whatever else you want to do before print

[gcode_macro END_PRINT]
gcode:
    G91 ; Relative positioning
    G1 E-3 F1800 ; Retract
    G1 F3000 Z3 ; Move up
    G90 ; Absolute pos.
    G1 X250 Y215; present print
    # Turn off bed, extruder, and fan
    M140 S0
    M104 S0
    M106 S0
    SAVE_IF_SET     ; SAVE_CONFIG if a mesh was probed in START_PRINT
    M84 ; Disable steppers

; This is where the magic happens:
; MAX_AGE is checked against the stored variable
; SAVE=1 can be used to force saving the mesh (restarts klipper, so
;        only for manual usage)
; FORCE_LEVEL=1 forces a mesh probe even if MAX_AGE is not reached
[gcode_macro LEVEL_BED_ADVANCED]
description: Levels the bed if the last leveling was MAX_AGE runs ago. Force leveling by setting FORCE
variable_save_at_end: 0
gcode:
  {% set max_age = params.MAX_AGE|default(10)|int %}
  {% set force_level = params.FORCE|default(0)|int %}
  {% set save = params.SAVE|default(0)|int %}

  ; load level_age from stored variables
  {% set svv = printer.save_variables.variables %}
  {% if "level_age" not in svv %} ; first run
    SAVE_VARIABLE VARIABLE=level_age VALUE={max_age}
    {% set level_age = 1 %}
  {% else %} ; load level_age and increment
    {% set level_age = svv.level_age %}
    SAVE_VARIABLE VARIABLE=level_age VALUE={level_age|int + 1}
  {% endif %}
  {action_respond_info("Bed mesh age is " + level_age|string) + "."}

  ; Level eventually
  {% if force_level or (level_age >= max_age|int) %}
    {action_respond_info("Bed mesh exceeded max age. Leveling...")}

    ; homing if not homed yet
    {% if 'xy' not in printer.toolhead.homed_axes %}
    G28
    {% endif %}

    BED_MESH_CALIBRATE
    {% if save %}
      SAVE_VARIABLE VARIABLE=level_age VALUE=1 ; reset counter
      SAVE_CONFIG
    {% else %}
      SET_GCODE_VARIABLE MACRO=LEVEL_BED_ADVANCED VARIABLE=save_at_end VALUE=1
    {% endif %}
  {% else %}
    {action_respond_info("Loading old bed mesh.")}
    BED_MESH_PROFILE LOAD=default
  {% endif %}

# runs SAVE_CONFIG if the g-code variable was set in start gcode
[gcode_macro SAVE_IF_SET]
gcode:
  {% if printer["gcode_macro LEVEL_BED_ADVANCED"].save_at_end == 1 %}
  {action_respond_info("Saving was requested - saving and restarting now.")}
  SAVE_VARIABLE VARIABLE=level_age VALUE=1
  SAVE_CONFIG
  {% endif %}
```
