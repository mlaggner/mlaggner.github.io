---
title: "Fixing automatic sleep on OSX 10.10+"
categories:
  - osx
tags:
  - osx
  - shellscript
summary:
  - image: osx.png
---
Automatic suspend to RAM (aka sleep or energy saving) has always been problematic on my wife's iMac: Her iMac did not go to sleep mode on idle or woke up accidentally and did not fell asleep again.

The source of this problem are assertions: these are used by multi media apps to keep the iMac awake while they play content. The Problem here is that not all apps work flawlessly with assertions. E.g. Spotify used to set such an assertion when it is opened (even if it is not playing any content - this has probably already been fixed); Chrome sets assertions when there is multimedia content in an tab (even not the active tab) and so on.

Recently I faced this problem again with Chrome - in one of the many open tabs such an assertion has been set. You can check this executing the command ```pmset -g assertions``` in the terminal. That looked like the following listing:

```
# pmset -g assertions
2017-02-16 16:48:14 +0100
Assertion status system-wide:
 BackgroundTask                 1
 ApplePushServiceTask           0
 UserIsActive                   0
 PreventUserIdleDisplaySleep    0
 PreventSystemSleep             0
 ExternalMedia                  0
 PreventUserIdleSystemSleep     1
 NetworkClientActive            0
Listed by owning process:
 pid 237(coreaudiod): [0x000ab1d00001012e] 00:00:05 PreventUserIdleSystemSleep named: "com.apple.audio.context568.preventuseridlesleep"
 Created for PID: 55482.
Kernel Assertions: 0x4=USB
 id=500  level=255 0x4=USB mod=28.01.17 11:46 description=EHC2 owner=AppleUSBEHCI
 id=501  level=255 0x4=USB mod=01.02.17 09:50 description=EHC1 owner=AppleUSBEHCI
```

In the past there has been the tool "Please Sleep" which could enforce the sleep mode on OSX, but that tool has been abandoned and does not work with OSX 10.10 onwards. So I decided to write a small shell script which does (almost) the same thing:

```
#!/bin/bash

# this will check how long before the system sleeps
# it gets the system sleep setting
# it gets the devices idle time
# it returns the difference between the two
#
# Note: if systemSleepTimeMinutes is 0 then the system is set to not go to sleep at all
#

systemSleepTimeMinutes=`pmset -g | grep "^[ ]*sleep" | awk '{ print $2 }'`

if [ $systemSleepTimeMinutes -gt "0" ]; then
   systemSleepTime=`echo "$systemSleepTimeMinutes * 60" | bc`
#    systemSleepTime=0
   devicesIdleTime=`/usr/sbin/ioreg -c IOHIDSystem | awk '/HIDIdleTime/ {print $NF/1000000000; exit}'`
   devicesIdleTime=`sed 's/,/\./' <<< $devicesIdleTime`
   secondsBeforeSleep=`echo "$systemSleepTime - $devicesIdleTime" | bc`
   echo "Time before sleep (sec): $secondsBeforeSleep"
   if (( $(bc <<< "$secondsBeforeSleep < 0.0") )); then
     echo "bye"
     `pmset sleepnow >/dev/null 2>&1 &`
   fi
else
   echo "The system is set to not sleep."
   exit 0
fi
```

 This shell script simply checks the configured sleep time from the settings and looks how long there has been no user interaction with the system. If the elapsed time since the last user interaction is high than the configured sleep time, this script puts the iMac to sleep.

 Now you just need a trigger for that script - and ```cron``` is the solution. Cron is installed on every OSX system and can be used by every user. Just add the following line to the users crontab (```crontab -e```) and cron triggers the script:

```
# min   hour    mday    month   wday    command
*/5    *       *       *       *       /bin/bash /Applications/_scripts/sleep.sh
```

I've chosen the interval to 5 minutes, since the iMacs energy saving settings are at 10 minutes. In the worst case, the iMac goes after 15 minutes to sleep.

The only drawback with this solution is that this script strictly enforces sleep - even you do not want it to be triggered (e.g. you watch a video file, large backup, ...).

In this case you simply have to increase the sleep time in the energy saving settings so that the iMac is wake long enough.
