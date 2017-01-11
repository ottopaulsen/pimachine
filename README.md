# Raspberry Pi with time machine for making backup of mac

The idea is to use an old hard drive and create a time machine backup solution for mac, using a Raspberry i. It shall be available on the wifi, only connected to power.

## Hardware

## Software

### Installation and configuration

Install Ubuntu Mate for Raspberry Pi by following [these instructions](https://www.raspberrypi.org/forums/viewtopic.php?f=56&t=143450).

I got an error when I tried to write to the memory card using `ddrescue: Direct disc access not available.`, so I used dd instead with this command:

```
sudo dd bs=1m if=ubuntu-mate-16.04-desktop-armhf-raspberry-pi.img of=/dev/rdisk2
```

Resize the memory card as described in the [Raspberry Pi instructions](https://ubuntu-mate.org/raspberry-pi/) so you can use the whole memory card.

Upgrade all software using "ubuntu-mate-welcome" (The welcome dialog in the GUI.)

Set up incoming ssh using the command `sudo apt-get install openssh-server`

Install public id with `ssh-copy-id <login>@<server>` in order to ssh directly in to the mate.

Avoid that the wifi turns off by `sudo vi /etc/pm/config.d/config`. Add the line

```
     SUSPEND_MODULES="iwlwifi"
```

Here is my `/etc/network/interfaces`:

```
auto lo wlan0
iface lo inet loopback
iface default inet dhcp
```

Also make sure that you have a file for your SSID (named as your SSID) in `/etc/NetworkManager/system-connections`. See [this post](https://ubuntu-mate.community/t/automatic-wifi-connection-on-startup/3216/3) for info.

Make sure that the file is rw for user:

```
sudo chmod 600 <filename>
```

A trick to regularly make sure the network is used is to create the file `/usr/local/bin/checkwlan` with this script:

```
#!/bin/bash    
wlan=$(/sbin/ifconfig wlan0 | grep inet\ addr -c)
if [ "$wlan" -eq 0 ]; then    
    /sbin/ifdown wlan0 && /sbin/ifup wlan0
else    
    echo interface is up    
fi
```

And call this regularly using crontab `sudo crontab -e`:

```
# Check wlan
10 * * * * /usr/local/bin/checkwlan 2>&1 | logger
```

### Configure hard drive

I used Ubuntu mate instructions for this...

Get hardware information: `sudo lshw -C disk`

My disk has data on it, so I don't want to format it now.

Make a mount point: `sudo mkdir /media/disk1`

Add this to the end of `/etc/fstab` so the drive is mounted at boot:

```
/dev/sda2    /media/disk1   ext4    defaults     0        2
```
That is if the filesystem is ext4. You can check with `sudo parted -l`







### Setting up time machine

Set up time machine following [these instructions](http://dae.me/blog/1660/concisest-guide-to-setting-up-time-machine-server-on-ubuntu-server-12-04/)

Here are my commands:

```
# Install:
sudo apt-get install netatalk avahi-daemon

# Add user(s):
sudo adduser otto
sudo adduser hilde
sudo adduser magnus

# Create directory for the backup:
sudo mkdir /home/otto/timemachine
sudo mkdir /home/hilde/timemachine
sudo mkdir /home/magnus/timemachine

sudo chown -R otto /home/otto/timemachine/
sudo chown -R hilde /home/hilde/timemachine/
sudo chown -R magnus /home/magnus/timemachine/

# Backup old configuration file:
sudo mv /etc/netatalk/AppleVolumes.default /etc/netatalk/AppleVolumes.default.old

# Create new configuration file
sudo vi /etc/netatalk/AppleVolumes.default

# Add the following lines to the file:
:DEFAULT: options:upriv,usedots
/home/otto/timemachine "Otto's Time Machine" options:tm volsizelimit:300000 allow:otto
/home/hilde/timemachine "Hilde's Time Machine" options:tm volsizelimit:300000 allow:hilde
/home/magnus/timemachine "Magnus' Time Machine" options:tm volsizelimit:300000 allow:magnus

# The restart the service
sudo service netatalk restart
```

Then on your mac, open Finder and press ⌘+K to connect to server. Then type `afp://<server-ip>/`. Log in with the username created above, and connect. You will be asked for password. If all goes well, you will open your time machine as a network drive.

Then open System Preferences->Time Machine, click “Select Disk…” and select your new server under “Available Disks”. Log in with the same credentials.




## Using Ubuntu Mate

You can turn the graphical interface off and on with these commands:

```
sudo graphical disable
sudo graphical enable
```

Update the kernel with

```
sudo rpi-update
```







