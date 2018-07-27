---
title: Probleme mit Adobe Flash 11.2 und NVIDIA
categories:
  - linux
tags:
  - linux
  - nvidia
summary:
  - image: tux.png  
---
Vor kurzem bin ich bei einem Update auf ein Problem gestoßen, dass Flash-Videos (z.B. von Youtube) mit fehlerhaften Farben abgespielt wurden.

Diese fehlerhafte Darstellung wirkte sich relativ ungünstig auf die Hautfarben aus: diese wurden Blau dargestellt:

<figure><a href="/images/2012/04/flash_player_11.2.jpg"><img src="/images/2012/04/flash_player_11.2.jpg" alt="Fehlerhafte Darstellung der Hautfarbe mit Flash Player 11.2 & NVIDIA"></a></figure>

Nach kurzer Recherche stellte sich heraus, dass das ein Problem im Zusammenspiel zwischen dem Adobe Flash Player 11.2 und dem binären NVIDIA Treiber ist. Es gibt aber glücklicherweise einen Workaround zu diesem Problem.

Dazu muss man nur 2 Dateien bearbeiten, einmal neu anmelden und schon funktioniert das Abspielen der Videos wieder wie gewünscht:

`/etc/profile.d/fix_flash.sh`

```
export VDPAU\_NVIDIA\_NO_OVERLAY=1
```

`/etc/adobe/mms.cfg`

```
EnableLinuxHWVideoDecode=1
```

Nach dieser Änderung funktioniert das Abspielen von Flash Videos wieder wie gewünscht.

***UPDATE 12.05.2012:***

heute musste ich die Änderungen zurücknehmen, da Flash andauernd gecrasht ist. Glücklicherweise stellte sich heraus, dass das Problem mit der blauen Haut nicht mehr auftritt.

Habe folgende Versionen von Flash und vom NVIDIA Treiber installiert:

flashplugin 11.2.202.235-1
nvidia 295.49-1

***UPDATE 13.05.2012:***

habe doch wieder ein Video gefunden (<https://www.youtube.com/watch?v=EKHZz-GY01w&feature=related>), wo die Darstellung falsch ist. Heißt es also mit blauer Haut zu leben oder in Kauf zu nehmen, dass der Flashplayer immer wieder crasht
