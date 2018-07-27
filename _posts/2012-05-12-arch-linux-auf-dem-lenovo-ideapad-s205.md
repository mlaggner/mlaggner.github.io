---
title: Arch Linux auf dem Lenovo IdeaPad S205
categories:
  - linux
tags:
  - linux
  - arch linux
summary:
  - image: tux.png  
---
Vor kurzem habe ich mein MSI Wind U100+ Netbook gegen ein Lenovo IdeaPad S205 getauscht. Die Grund für den Tausch war hauptsächlich die Displaygröße von 10,2" und die daraus resultierende Auflösung von 1024x600. In letzter Zeit habe ich das Netbook immer mehr aktiv benützt, aber mit dieser Auflösung konnte man nicht produktiv arbeiten. Zusätzlich war der Prozessor (ein Singlecore Atom) und die integrierte Grafikkarte fast zu schwach für die GNOME Shell.

Darum habe ich mich für das Lenovo IdeaPad S205 als Alternative entschieden. Mit 11,6" Displaygröße, einer Auflösung von 1366x768 und einem neuen AMD-450 Prozessor schien mir das als ideales Netbook mit einem vernünftigen Preis/Leistungs Verhältnis.

Als Betriebssystem kam für mich nur mehr Archlinux in Frage, da das auf dem MSI Wind und auf meinem PC einwandfrei läuft. Bei der Installation habe ich mich für die 32bit Version entschieden (obwohl ich 4GB RAM im Netbook habe), da ich am PC schon einige Probleme mit den 32bit Libs für Citrix hatte. Daher wollte ich hier direkt schon auf Nummer sicher gehen - vor allem weil beim Netbook die 64bit Version keinen nennenswerten Vorteile brachte.

Bei der Installation folgte ich dem äußerst gut geschriebenem Wiki-Artikel [Beginner's Guide](https://wiki.archlinux.org/index.php/beginners'_guide) für die Installation des Grundsystems. Beim ersten Schritt der Installation habe ich nur das Grundsystem ohne X/GNOME installiert. Einzig VIM habe ich zusätzlich installiert, da das mein Favorit der Editoren ist.

Nach dem ersten Neustart habe ich mir erstmal die (von Gentoo und Ubuntu) gewohnten Tools nachinstalliert, die ich ich öfters benötige:

  * net-tools (für ifconfig) - **NOTIZ**: wie ich gerade gelesen habe sollte man die net-tools nicht mehr verwenden, da sie nicht mehr gewartet werden
  * wireless-tools und wpa_supplicant (für WLAN)

Danach habe ich mit der Installation von GNOME angefangen. Dazu muss zuerst X installiert werden. Folgende Pakete habe ich installiert:

```
pacman -S xorg-server xorg-xinit xorg-server-utils mesa mesa-demos xf86-video-ati
```

Anschließend habe ich in der Datei `/etc/X11/xorg.conf.d/10-evdev.conf` die Einstellung für deutsches Tastaturlayout im GDM eingestellt:

```
Section "InputClass"
Identifier "evdev keyboard catchall"
MatchIsKeyboard "on"
MatchDevicePath "/dev/input/event*"
Driver "evdev"
Option "XkbLayout" "de"
EndSection
```

Danach kam noch `dbus` dazu, da GNOME auf dbus zurückgreift (und danach natürlich auch in der `/etc/rc.conf` als Dienst eintragen):

```
pacman -Sy dbus
rc.d start dbus
vim /etc/rc.conf
```

Da jetzt alle Abhängigkeiten für GNOME installiert waren, habe ich das GNOME Komplettpaket installiert:

```
pacman -S gnome gnome-extra gdm
```

Wenn die Installation von GNOME fertig ist, wird noch `gdm` als Dienst in der `/etc/rc.conf` eingetragen und beim nächsten Reboot sollte dann schon der Login via gdm möglich sein. Spätestens nach dem ersten Einloggen als "normaler" User fiel mir auf, dass `sudo` noch nicht installiert war, was ich daraufhin sofort nach holte:

```
pacman -Sy sudo
```

und anschließend wird noch in mit `visudo` und `PolicyKit` eingestellt, dass normale User auch Root-Rechte erlangen dürfen. Mit `visudo` habe ich folgende Einstellung vorgenommen:

```
## Uncomment to allow members of group wheel to execute any command
%wheel ALL=(ALL) ALL
```

und für `PolicyKit` habe ich die Datei `/etc/polkit-1/localauthority.conf.d/50-localauthority.conf` folgendermaßen geändert:

```
[Configuration]
AdminIdentities=unix-group:wheel
```

Es war an dieser Stelle somit alles wichtige für einen reibungslosen Betrieb eingestellt, jedoch stellte ich noch ein paar Probleme fest:

  * Die Systemeinstellungen in der GNOME Shell ließen sich nicht starten. Nach einiger Recherche fand ich heraus, dass es da einen Bug mit den Opensource Treibern von NVIDIA und ATI gab. Abhilfe schafft der Aufruf über die Console mit folgendem Befehl:

```
MALLOC\_CHECK\_=1 gnome-control-center
```

Da ich aber keine Lust hatte, jedesmal die Einstellungen über das Terminal mit diesem Befehl zu starten, fügte ich in der Datei `~/.xprofile` folgende Zeile hinzu:

```
export MALLOC\_CHECK\_=1
```

Somit wird, nach erneutem Login, das GNOME Control Center auch über die GNOME Shell erfolgreich gestartet.

  * Das Touchpad "zitterte", und somit konnte man recht schlecht mit dem Mauszeiger zielen. Anfangs nahm ich das so hin, aber dann fand ich heraus, dass man ein spezielles Paket installieren muss damit das Touchpad richtig funktioniert:

```
pacman -Sy xf86-input-synaptics
```

Und in der Datei `/etc/X11/xorg.conf.d/10-synaptics.conf` machte ich folgende Einstellungen:

```
Section "InputClass"
Identifier "touchpad catchall"
Driver "synaptics"
MatchIsTouchpad "on"
MatchDevicePath "/dev/input/event*"
Option "TapButton1" "0"
Option "TapButton2" "0"
Option "TapButton3" "0"
Option "VertEdgeScroll" "on"
Option "VertTwoFingerScroll" "on"
EndSection
```

Damit funktionierte das Touchpad sehr gut: die Empfindslichkeit war super, scrollen mit 2 Fingern ist möglich, und Tippen aufs Touchpad wird nicht als Click interpretiert.

  * Da die vertikale Auflösung auch bei diesem Netbook sehr stark beschränkt ist, habe ich noch [Maximus](http://aur.archlinux.org/packages.php?ID=22071) (für das Verstecken des oberen Fensterrahmens bei maximierten Fenstern) installiert.

Somit war das System fertig eingerichtet. Später habe ich die GNOME Shell dann noch optisch etwas aufpoliert (mit meinem Zukitwo Cupertino Thema + Extensions, welche ich auch in dem Beitrag zum Thema erwähnt habe) und seitdem ist das Netbook ein treuer und zuverlässiger Begleiter 😉

Abschließend noch ein Auszug an Programmen bzw. Paketen, die ich am Netbook extra installiert habe:

  * Chromium (meint Favorit bei den Browsern)
  * Clementine (sehr guter Musikplayer - ist zwar mit Qt programmiert, integriert sich aber super mit der GNOME Shell)
  * VLC (der Mediaplayer und zwar auf jedem OS)
  * yaourt (nettes Programm zum Installieren von Pakages aus dem AUR Repository)
  * gnome-tweak-tool (ein must have für die GNOME Shell)
  * gtk-engine-murrine gtk-engine-unico ttf-dejavu (Voraussetzungen für das GTK Thema Adwaita Cupertino)
  * Adwaita Cupertino
  * Faenza + Faenza Cupertino
  * Zukitwo Cupertino
