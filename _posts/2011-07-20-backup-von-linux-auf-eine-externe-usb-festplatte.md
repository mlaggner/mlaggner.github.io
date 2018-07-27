---
title: Backup von Linux auf eine externe USB-Festplatte
categories:
  - linux
tags:
  - linux
  - backup
summary:
  - image: tux.png  
---
Da ja bekanntlich ein NAS kein Backup ist und keines ersetzen kann, musste ich mir überlegen, wie ich die Daten meines NAS sichern kann.

Mein NAS besteht aus einem alten AMD System (Athlon X2 BE-2350, 2 GB RAM, 3 x 1TB HDD im RAID5 Verbund, 160 GB HDD - OS, GBit NIC), auf dem Ubuntu 10.04 mit einer Serverinstallation betrieben wird. Die Nutzdaten des NAS liegen in einem RAID5 Verbund (/dev/md0 gemountet auf /mnt/NAS) und sollen in in regelmäßigen Abständen auf eine externe USB-Festplatte gesichert werden.

Dazu habe ich mir eine externe 2TB USB-Festplatte gekauft, auf dieser eine Partition erstellt und mit EXT4 formatiert. Anschließend musste ich der Partition noch ein Label vergeben, damit diese bei jedem Mount automatisch den gleichen Mountpoint bekommt (damit muss kein Eintrag in der `/etc/fstab erzeugt` werden):

```
NAS:~# sudo parted
(parted) select /dev/sde
(parted) unit %
(parted) mkpartfs primary ext4 0 100%
(parted) quit
NAS:~# sudo e2label /dev/sde1 Backup
```
Somit wurde der Speicherplatz auf der USB-Festplatte eingerichtet und es wird bei jedem Mount der gleiche Mountpoint zugewiesen. Damit kann das Script für die rsync Sicherung von /mnt/NAS eingerichtet werden. Ich habe dazu das Script von [wiki.ubuntuusers.de](http://wiki.ubuntuusers.de/skripte/Backup_mit_RSYNC) verwendet. Für meinen Server sieht dieses Script folgendermaßen aus:

```
#!/bin/bash
# Simple backup with rsync
# local-mode

SOURCES="/mnt/NAS/"
TARGET="/media/Backup/"
RSYNCCONF="-delete"
MOUNTPOINT="/media/Backup"
#PACKAGES=1
#MONTHROTATE=1
#MAILREC="user@localhost"
#SSHUSER="root"
#SSHPORT=22
#FROMSSH="clientsystem"
#TOSSH="backupserver"

### do not edit ###
LOG=$0.log
MAILSUBJECT="Backup $LOG"
MOUNT="/bin/mount"; FGREP="/bin/fgrep"; SSH="/usr/bin/ssh"
LN="/bin/ln"; ECHO="/bin/echo"; DATE="/bin/date"; RM="/bin/rm"
DPKG="/usr/bin/dpkg"; AWK="/usr/bin/awk"; MAIL="/usr/bin/mail"
CUT="/usr/bin/cut"; TR="/usr/bin/tr"; RSYNC="/usr/bin/rsync"
LAST="last"; INC="-link-dest=../$LAST"

$DATE > $LOG

if [ -n "$PACKAGES" ] && [ -z "$FROMSSH" ]; then
  $ECHO "$DPKG -get-selections | $AWK '!/deinstall|purge|hold/'|$CUT -f1 | $TR 'n' ' '" >> $LOG
  $DPKG -get-selections | $AWK '!/deinstall|purge|hold/'|$CUT -f1 |$TR 'n' ' ' >> $LOG 2>> $LOG
fi

MOUNTED=$($MOUNT | $FGREP "$MOUNTPOINT");

if [ -z "$MOUNTPOINT" ] || [ -n "$MOUNTED" ]; then
  if [ $MONTHROTATE ]; then
    TODAY=$($DATE +%d)
  else
    TODAY=$($DATE +%y%m%d)
  fi

  if [ "$SSHUSER" ] && [ "$SSHPORT" ]; then
    S="$SSH -p $SSHPORT -l $SSHUSER";
  fi

  for SOURCE in $($ECHO $SOURCES) do
    if [ "$S" ] && [ "$FROMSSH" ] && [ -z "$TOSSH" ]; then
      $ECHO "$RSYNC -e "$S" -avR $FROMSSH:$SOURCE $RSYNCCONF $TARGET$TODAY $INC" >> $LOG
      $RSYNC -e "$S" -avR $FROMSSH:$SOURCE $RSYNCCONF $TARGET$TODAY $INC >> $LOG 2>> $LOG
    fi

    if [ "$S" ] && [ "$TOSSH" ] && [ -z "$FROMSSH" ]; then
      $ECHO "$RSYNC -e "$S" -avR $SOURCE $RSYNCCONF $TOSSH:$TARGET$TODAY $INC " >> $LOG
      $RSYNC -e "$S" -avR $SOURCE $RSYNCCONF $TOSSH:$TARGET$TODAY $INC >> $LOG 2>> $LOG
    fi

    if [ -z "$S" ]; then
      $ECHO "$RSYNC -avR $SOURCE $RSYNCCONF $INC $TARGET$TODAY" >> $LOG 2>> $LOG;
      $RSYNC -avR $SOURCE $RSYNCCONF $INC $TARGET$TODAY >> $LOG 2>> $LOG
    fi

  done

  if [ "$S" ] && [ "$TOSSH" ] && [ -z "$FROMSSH" ]; then
    $ECHO "$SSH -p $SSHPORT -l $SSHUSER $TOSSH $LN -nsf $TARGET$TODAY $TARGET$LAST" >> $LOG
    $SSH -p $SSHPORT -l $SSHUSER $TOSSH "$LN -nsf $TARGET$TODAY $TARGET$LAST" >> $LOG 2>> $LOG
  fi

  if ( [ "$S" ] && [ "$FROMSSH" ] && [ -z "$TOSSH" ] ) || ( [ -z "$S" ] ); then
    $ECHO "$LN -nsf $TARGET$TODAY $TARGET$LAST" >> $LOG
    $LN -nsf $TARGET$TODAY $TARGET$LAST >> $LOG 2>> $LOG
  fi
  $DATE >> $LOG
else
  $ECHO "$MOUNTPOINT not mounted" >> $LOG
fi

if [ -n "$MAILREC" ];then
  $MAIL -s "$MAILSUBJECT" $MAILREC < $LOG
fi
```

Damit brauche ich nur meine externe USB-Festplatte an mein NAS anschließen (es wird automatisch unter /media/Backup gemountet) und anschließend das Backupscript starten. Durch Rsync werden nur geänderte Dateien geschrieben und da das Dateisystem auf der USB-Festplatte ext4 ist (und dadurch auch Hardlinks unterstützt werden), habe ich pro "Backup" ein Verzeichnis mit allen Dateien - Dateien, welche in mehreren Backups vorkommen werden durch die Hardlinks aber nur einmal physisch auf der Platte gesichert.
