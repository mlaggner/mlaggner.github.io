---
title: Programm mit Superuser-Rechten unter OSX starten
categories:
  - macOS
tags:
  - macOS
summary:
  - image: apple.png  
---
Manchmal ist es in Mac OSX nötig, ein Programm mit Superuser-Rechten zu starten. Wenn das Programm nicht von alleine dieses Recht anfordert, steht man vor einem Problem. Es ist dennoch über einen kleinen Umweg möglich, ein Programm als Superuser zu starten. Dazu sind folgende Schritte nötig:<!--more-->

  1. Starten des Terminals (am Schnellsten über Spotlight -> Terminal)
  2. Im Terminal mit `sudo -s` auf den Superuser wechseln
     <figure><a href="/images/2011/03/sudo.png"><img src="/images/2011/03/sudo-thumb.png"></a></figure>
  3. Das gewünschte Programm im Finder suchen und über das Kontextmenü (rechte Maustaste) "Paketinhalt zeigen" auswählen.
     <figure><a href="/images/2011/03/context_menu.png"><img src="/images/2011/03/context_menu-thumb.png"></a></figure>
     Daraufhin wird ein weiteres Finder-Fenster geöffnet, in dem man den Paketinhalt des gewünschten Programms findet.
     <figure><a href="/images/2011/03/package_contents.png"><img src="/images/2011/03/package_contents-thumb.png"></a></figure>
  4. Normalerweise befindet sich das eigentliche Programm im Pfad Contents/MacOS - wenn man im Finder dorthin navigiert müsste mindestens ein Ausführbares Programm zu sehen sein.
     <figure><a href="/images/2011/03/drag_and_drop.png"><img src="/images/2011/03/drag_and_drop-thumb.png"></a></figure>
  5. Dieses Programm zieht man per Drag and Drop auf das Terminal-Fenster. Somit wird der komplette Pfad des Programmes im Terminal hinterlegt und daraufhin kann man es direkt vom Terminal aus als Superuser starten.
