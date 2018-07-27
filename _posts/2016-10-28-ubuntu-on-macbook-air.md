---
title: "Ubuntu Gnome 16.04 on a Macbook Air 2014"
categories:
  - linux
tags:
  - linux
  - ubuntu
summary:
  - image: tux.png
---
 I've recently got this Macbook as a replacement for my Macbook Pro 2012. After falling in love with Ubuntu on a Macbook (great OS on great hardware!), I've decided to install Ubuntu Gnome 16.04 along with Mac OSX El Capitan on this Macbook.

First of all you should download the Ubuntu flavor which suits your need. I decided for [Ubuntu Gnome](https://ubuntugnome.org/) because I use Gnome for most of the time. Simply download the installer image (I used 16.04.1) and [create a bootable USB](https://www.ubuntu.com/download/desktop/create-a-usb-stick-on-mac-osx) stick off it.

I also had to shrink the OSX partition to create space for the Ubuntu installation. I've got a model with 256GB SSD, so I decided to give OSX 100GB and the remaining space for Ubuntu. To repartition the SSD you simply have to reboot your Macbook and press `cmd + r` while the boot sound appears. This will boot the Macbook into recovery mode which offers the `disk utility`. Just start the `disk utility`, select the whole SSD and press `Partition`. In the following screen you can resize the disk space for your OSX Partition.

After resizing the partiotion Ubuntu is ready to get installed. Reboot your Macbook, inser the USB stick and press `alt` while the sound appears and you should see two bootable devices (your SSD and the USB stick). Choose the USB stick, boot into the graphical live session and start the installation there. Just click through the install wizard _except_ the option for the install destination.

The installer offered me to install Ubuntu instead of OSX, but I used the expert options to split the partition for Ubuntu into two partitions (4GB for swap, the rest for /) and continued.

After Ubuntu has been installed there was one little thing not working: the display brightness after susped was either 0% or 100%. A fix for that can be obtained at [GitHub](https://github.com/patjak/mba6x_bl). After installing this fix everything worked like a charm.
