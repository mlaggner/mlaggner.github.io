---
title: "Install Manjaro on a Lenovo Yoga 530 14-ARR (AMD version)"
categories:
  - linux
tags:
  - linux
summary:
  - image: tux.png
---

Today I got my brand new Lenovo Yoga 530 14-ARR (AMD version). After finishing the Windows 10 setup, I decided to give [Manjaro KDE](https://manjaro.org/get-manjaro/) a try. Setting up Linux on this device is not as easy as on my Macbook Air, but finally I managed to set up everything.

First of all you should get the installation media for your desired linux and prepare an USB stick for the installation.

## Prepare a partition for Linux

To have some space for Linux, I've reduced the Windows partition to about 120GB so that Linux has about 120GB too. To do that you have to do the following steps:

* Right click Computer on the desktop, choose Manage.
* Select Storage > Disk Management.
* Right click the partition that you want to reduce, select Shrink Volume.
* Edit the proper size for the new partition, then click Shrink.

After preparing the partition you are ready to start the installation.

## Turn off secure boot

To be able to boot off the prepared USB stick (and Linux later on), you have to disable secure boot in the BIOS. This can be done with the following steps:

* Press F2 at boot to enter the BIOS
* Go to Security > Secure Boot
* Change Secure Boot to "Disabled"
* Press F10 to "Save and Exit"

## Boot from the USB stick and install Linux

To boot from the USB stick rather than the Windows partition, you have to press `F12` at boot time. Choose the prepared USB stick and boot into the Linux installer (Manjaro KDE in my case).

Unfortunately not everything (touchpad and wifi) worked out of the box in the live environment, so I had to insert an USB hub, a USB to LAN adapter ([CSL USB 3.0 3-Port Hub incl. Gigabit Ethernet LAN](https://amzn.to/2SgfQWL) in my case) and a normal USB mouse.

Having troubles with the Wifi adapter caused the installer to crash, but I found a [fix](https://github.com/manjaro/mhwd/issues/50) to prevent this crash: edit the file `/usr/lib/calamares/modules/mhwdcfg/main.py` and remove the lines

```
for id in self.identifier['net']:
  self.configure(b, id)
```

and you are able to finish the installation without a problem. Manjaro has been installed to the spare partition, the bootloader has been installed to the EFI partition and a boot entry for Windows has been created.


## Fix for the Wifi adapter

After you have finished the installation, you are able to reboot into the installed system. In my case Wifi and the touchpad still did not work (even after a system upgrade).

In Manjaro/Arch Linux there is a package in AUR which provides the proper driver for the Wifi module: [rtl8821ce-dkms-git](https://aur.archlinux.org/packages/rtl8821ce-dkms-git/). To compile this driver I had to install the kernel headers too.

This module will get automatically loaded at boot time, but another module is preventing it from working. You should blacklist the module `ideapad_laptop` to get it working:

```bash
sudo tee /etc/modprobe.d/ideapad.conf <<< "blacklist ideapad_laptop"
```

## Fix for the touchpad (and touchscreen)

Finding a fix for the touchpad was rather hard. But finally I found a GitHub repository with a working driver for the touchpad:  [i2c-amd-mp2](https://github.com/Syniurge/i2c-amd-mp2). You can clone this repo, build and install the module with the commands

```bash
git clone https://github.com/Syniurge/i2c-amd-mp2.git
cd i2c-amd-mp2
sudo ./dkms-install.sh
```

With this module your touchpad will begin to work, but in my case I needed to create a config for xorg to enable the [multi finger click](https://wayland.freedesktop.org/libinput/doc/1.11.3/clickpad_softbuttons.html). To do this, create the file `/usr/share/X11/xorg.conf.d/52-elantech-touchpad.conf` with the following content:

```
Section "InputClass"
       Identifier "libinput touchpad catchall"
       MatchIsTouchpad "on"
       MatchDevicePath "/dev/input/event*"
       Driver "libinput"
       Option "ClickMethod" "clickfinger"
EndSection
```

You find some good information for libinput configuration at the [Arch Linux Wiki](https://wiki.archlinux.org/index.php/Libinput).

With these tweaks I have a fully working Lenovo Yoga 530 to work with!
