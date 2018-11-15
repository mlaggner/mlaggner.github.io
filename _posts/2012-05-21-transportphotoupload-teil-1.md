---
title: 'TransportPhotoUpload - Teil 1'
categories:
  - programming
tags:  
  - android
  - html/javascript
  - sap
summary:
  - image: sap.gif  
---
Im Rahmen der Evaluierung von mobilen Geräten im Zusammenspiel mit unserem SAP-Systemen (hauptsächlich R3) wurde ich mit einem Projekt betraut, wo Fotos von einem mobilen Endgerät über das lokale Netzwerk in SAP gesendet werden soll.

Im Detail war die Aufgabenstellung folgende: In der Verladung werden bis jetzt Fotos von jedem Transport geschossen (zwecks Dokumentation) und diese dann über einen PC auf einen speziellen Fileshare gespielt. Dies wird alles manuell gemacht und ist somit auch fehleranfällig.

Die Idee, diesen Prozess zu optimieren, wäre folgende: der Mitarbeiter an der Verladung soll über ein mobiles Gerät den Barcode beim Verladeschein scannen können (somit hat er die Referenz zum Transport). Mit diesem Gerät soll er anschließen die Fotos des Transportes (LKW, Waggon) schießen und dann online direkt ins SAP, als Anhang zu dem jeweiligen Beleg, schicken können.

Durch die Beschreibung merkt man schon, dass dieses Projekt recht komplex ist und auch viele Teilbereiche umspannt. Daher werde ich die einzelnen Teile des Projektes ein extra Beiträgen beschreiben. Hier im ersten Teil werde ich auf die grundsätzlichen Aufbau eingehen.

![Ablauf_TransportPhotoUpload](/images/2012/05/ablauf_tpu.gif){: .align-center}

1. Der Mitarbeiter an der Verladung scannt den Verladeschein
2. Der Mitarbeiter an der Verladung macht Fotos von dem LKW/Waggon
3. Über das lokale Netzwerk werden die Fotos an den SAP Netweaver Gateway 2.0 gesendet
4. Der SAP Netweaver Gateway 2.0 leitet diese Fotos weiter ans R3 System, welches die Fotos als Anlagen zum Transportbeleg sichert

Durch den generellen Ablauf des Prozesses standen schon einige Komponenten fest, die benötigt werden:

* Zugriff aufs SAP R3 System in Form eines RFC Bausteins
* SAP Netweaver Gateway 2.0 (im Zuge einer Testinstallation vorhanden)
* WLAN am Verladeort (war bereits vorhanden)

Offen war zu diesem Zeitpunkt nur das mobile Endgerät. Da wir ohnehin ein ähnliches Setup im Zuge eines anderen Projektes testen wollten, entschieden wir uns auch hier für ein Smartphone. Doch auch die Entscheidung, welches Smartphone gab uns in diesem Fall noch einiges an Überlegung auf (abgesehen von den bereits vorher abgeklärten minimalen Anforderungen):

* Android: Programmiert wird mit Java (Know-How vorhanden); leicht zu testen/debuggen; leider nicht mit der Firmenpolitik (Policies) zu vereinen
* iOS: Programmiert wird mit Objective C (Know-How nicht vorhanden); ein Mac mit OSX wird benötigt; umständlich zu testen/einzurichten
* Blackberry: Programmiert wird mit Java (Know-How vorhanden); Hardware leider nicht am neuesten Stand

Wir als Entwickler würden da ja gerne Android als Plattform bevorzugen, was aber leider von der Firmenpolitik her nicht realisierbar ist. Die Alternative dazu wäre iOS, aber dazu bräuchten wir einen extra Mac - wo wir auf der anderen Seite wieder unsere gewohnten Arbeitsmittel nicht zur Verfügung haben (Outlook, SAP usw.). Zusätzlich müssten wir für das Endgerät extra die Developerlizenz erwerben, damit wir ohne Umwege über den Appstore unser Programm darauf spielen können. Blackberry schied von Anfang an komplett aus, da es für so eine Anwendung nicht die ideale Umgebung ist.

Daher entschieden wir uns für das Framework [PhoneGap](http://phonegap.com/) welches es ermöglicht, eine Applikation nur mit HTML5 und Javascript zu schreiben und dieses dann, verpackt als native Anwendung, auf die jeweilige Plattform zu verteilen. Die Idee dahinter ist vielversprechend - so konnten wir auch im Browser testen, ein Android als Testgerät verwenden und die Applikation schlussendlich auf ein Blackberry ausliefern.

Somit waren die Grundsteine gelegt, wir wussten, welche Teile der Infrastruktur benötigt wurden, welche Umgebung bzw. Sprache zum Entwickeln verwendet wird und auf welchen mobilen Geräten getestet wird.

Im nächsten Teil werde ich auf den Aufbau und die Kommunikation zwischen dem SAP Netweaver Gateway 2.0 und SAP R3 eingehen. Anschließend wird in weiteren Posts die Applikation und Kommunikation zwischen dem mobilen Gerät und dem SAP Netweaver Gateway 2.0 und die damit verbundenen Probleme erläutert.
