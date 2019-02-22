# Using Raspberry Pi as a Print Server

## 1. Preparing the Raspberry Pi

Save Raspbian image to SD and enable SSH:

```sh
// add Etcher repository:
$ echo 'deb https://deb.etcher.io stable etcher' | sudo tee /etc/apt/sources.list.d/balena-etcher.list
$ sudo apt-key adv --keyserver keyserver.ubuntu.com --recv-keys 379CE192D401AB61
$ sudo apt-get update
$ sudo apt-get install -y balena-etcher-electron

// open Etcher and save image to sd card in /dev/mmcblk0
$ balena-etcher-electron

// unplug and plug the sd card to get the path to `boot` partition
$ df -h
...
tmpfs           379M   56K  379M   1% /run/user/1001
/dev/mmcblk0p1   44M   23M   22M  51% /media/roger/boot
/dev/mmcblk0p2  1.7G  950M  621M  61% /media/roger/rootfs

// enables ssh
$ touch /media/roger/boot/ssh
```

Insert sd card into your Raspberry Pi, connect it to your network and boot it. 




Update RPi
```sh
$ sudo apt-get -y update
$ sudo apt-get -y upgrade
$ sudo reboot
```
## 2. Pre-configure Raspberry Pi

Change password, resize root partition
```sh
$ sudo raspi-config
```

```sh
///  'dpkg-reconfigure tzdata' if you wish to change it.
```

Set static IP address to `eth0` and set a proper `hostname`
```sh

```

## 3. Install and configure the Print Server
```sh
$ sudo apt-get install -y cups
$ sudo usermod -a -G lpadmin pi
$ sudo cupsctl --remote-any
$ sudo /etc/init.d/cups restart
``` 

## 4. Setting up SAMBA for the Print Server

```sh
$ sudo apt-get install -y samba
$ sudo nano /etc/samba/samba.conf   

// in the [printers] section and change the; guest ok = no to guest ok = yes
// in  the [print$] printer driver section, change the; read only = yes to read only  = no

$ sudo /etc/init.d/samba restart
```

## 5. References


### HP PhotoSmart C4280 Drivers:
```sh
- http://www.openprinting.org/printer/HP/HP-PhotoSmart_C4200
- hplip drivers: https://developers.hp.com/hp-linux-imaging-and-printing
- http://www.openprinting.org/printer/HP/HP-Photosmart_C4280
```

### CUPS and Samba Printing:
- http://www.penguintutor.com/linux/printing-cups
- https://pimylifeup.com/raspberry-pi-print-server/
- https://circuitdigest.com/microcontroller-projects/raspberry-pi-print-server
- https://github.com/diadzine/ansible-role-rpi-hp-mfp
```