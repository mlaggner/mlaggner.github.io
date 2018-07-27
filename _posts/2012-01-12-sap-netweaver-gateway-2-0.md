---
title: SAP Netweaver Gateway 2.0
categories:
  - programming
tags:  
  - sap
  - abap
summary:
  - image: sap.gif
---
Vor kurzem hatte ich die Möglichkeit den SAP Netweaver Gateway 2.0 im Zuge einer Evaluierung von Mobilen Anwendungen zu testen.

Das System wurde als Testversion vom SDN heruntergeladen und anschließend von der Basis installiert und eingerichtet. Schlussendlich konnte ich mit dem Standarduser _DEVELOPER_ und dem Passwort _ch4ngeme_ im Backend einloggen und mich dort einmal umsehen.

Schon beim ersten Versuch, mir das SAP-Standardbeispiel in der SE80 anzusehen, kamen die ersten Probleme: Anscheinend sind die Tools für das Erstellen von GW Model und GW Consumer nicht vollständig übersetzt, sodass ich mich auf Englisch anmelden musste.

Also nahm ich das erste Beispiel von SAP zur Hand ([es gibt zu diesem Thema sehr viele und auch gute Beispiele!](http://www.sdn.sap.com/irj/sdn/index?rid=/webcontent/uuid/c0a638d0-8478-2e10-8eb4-f157b64fb221)) und versuchte einen RFC Baustein von unserem R3-Testsystem anzusprechen. Dazu habe ich [dieses Howto](http://www.sdn.sap.com/irj/sdn/go/portal/prtroot/docs/library/uuid/a0b24f72-d4e2-2e10-ce9e-dc24975ddd80) verwendet. Nachdem ich einen einfachen RFC Baustein erstellt hatte, der mir nur die Tabelle _VBAK_ im R3-System liest und  mir die Basis die RFC Verbindung zum R3-Testsystem eingestellt hatte, musste ich noch im Customizing (SPRO) im Pfad _SAP Customizing Implementation Guide->SAP NetWeaver->Gateway->Connection Settings->SAP NetWeaver Gateway to SAP System->Manage SAP System Aliases_ einen System Alias für das R3-Testsystem einrichten. Als dieses erledigt war, konnte ich laut Dokumentation mit der Erstellung des GW Data Models fortfahren.

Um das GW Data Model zu erstellen, sind 2 Wege möglich:

* direkt durch die Transaktion /IWFND/GWO_GEN (beim Testsystem vom SDN im Menü)
* über die SE80
<figure><a href="/images/2012/01/create_data_model_1.png"><img src="/images/2012/01/create_data_model_1.png" alt="Erstellung des GW Data Model - Schritt 1"></a></figure>

Die Einrichtung des GW Data Models verlief im Grunde ohne Probleme (sofern man keine Strings in der RFC Schnittstelle verwendet) und der RFC Baustein konnte Minuten später über den SAP Netweaver Gateway 2.0 aufgerufen werden (QUERY).

Jetzt stand ich vor dem nächsten Problem: Ich wollte mit der Operation QUERY nicht immer alle Aufträge aus der _VBAK_ lesen, sondern über Parameter die Selektionsoptionen mitgeben. Dazu stand im Howto, dass man über das OData Protokoll mittels dem Parameter _$filter_ Filterwerte mitgeben kann. Dazu muss der RFC-Baustein um eine Schnittstelle für die Selektionskriterien erweitert werden. Ich habe dazu den Typ _BAPILAWRGE_ verwendet:

```
FUNCTION ztest_get_vbak.
*"------------------------------------------------
*"Lokale Schnittstelle:
*" EXPORTING
*" VALUE(ET_VBAK) TYPE VBAK_T
*" TABLES
*" IT_SELOPTS STRUCTURE BAPILAWRGE
*"------------------------------------------------

DATA: lv_debug TYPE c.

DATA: lr_kunnr TYPE RANGE OF kunnr,
      lr_vkorg TYPE RANGE OF vkorg.

DATA: ls_selopt TYPE bapilawrge,
      ls_kunnr LIKE LINE OF lr_kunnr,
      ls_vkorg LIKE LINE OF lr_vkorg.

* WHILE lv_debug IS INITIAL.
*
* ENDWHILE.

" Selektionsparameter
LOOP AT it_selopts INTO ls_selopt.
  CASE ls_selopt-parameter.
    WHEN "KUNNR".
      MOVE-CORRESPONDING ls_selopt TO ls_kunnr.
      APPEND ls_kunnr TO lr_kunnr.

    WHEN "VKORG".
      MOVE-CORRESPONDING ls_selopt TO ls_vkorg.
      APPEND ls_vkorg TO lr_vkorg.

    WHEN OTHERS.

  ENDCASE.
ENDLOOP.

CHECK lr_kunnr IS NOT INITIAL.
CHECK lr_vkorg IS NOT INITIAL.

" Aufträge lesen
SELECT *
  FROM vbak
  INTO TABLE et_vbak
  WHERE kunnr IN lr_kunnr
    AND vkorg IN lr_vkorg.

ENDFUNCTION.
```

Dieser Baustein liest für alle Selektionsparameter (_KUNNR_ und _VKORG_ in diesem Fall) die Werte aus, baut sich Rangetables auf und selektiert am Schluss die Tabelle _VBAK_. Die auskommentierte Schleife _WHILE ... ENDWHILE_ habe ich zum RFC-debuggen eingebaut. Muss man den Baustein bei einem RFC-Aufruf debuggen, muss man das Kommentar der 2 Zeilen löschen und dann bleibt der Baustein in einer Endlosschleife. Somit kann man sich über die Transaktion SM50 den Prozess zum debuggen holen, in Debugger die Variable _LV_DEBUG_ auf ungleich space ändern und hat die Möglichkeit, den Baustein komplett zu debuggen.

Anschließend konnte ich beim Ändern des GW Data Models das Mapping der Selektionsparameter via Rangetable vornehmen:

<figure><a href="/images/2012/01/create_data_model_2.png"><img src="/images/2012/01/create_data_model_2.png" alt="Mapping der Rangetable für die Selektionsoptionen"></a></figure>

Jetzt werden auch über den Parameter _$filter=kunnr EQ "0000010016" AND vkorg EQ "0100"_ nur die entsprechenden Kundenaufträge gefiltert.

Somit war mein erster Test mit einem RFC-Auftrag im R3-Testsystem über den SAP Netweaver Gateway 2.0 erfolgreich und ich kann mich auf weitere Themen von diesem Projekt stürzen.

UPDATE: wie sich herausgestellt hat, sind Strings bei RFC Aufrufen zwischen dem SAP Netweaver Gateway 2.0 und SAP R3 prinzipiell kein Problem. Es darf nur kein Keyfeld (Keyfeld beim Mapping des RFC Bausteins) als String verwendet werden. Übertragen von Daten per String ist jedoch möglich (XSTRING geht jedoch nicht).
