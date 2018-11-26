---
title: 'TransportPhotoUpload - Teil 2'
categories:
  - programming
tags:
  - sap
  - abap
summary:
  - image: sap.gif
---
Im ersten Teil habe ich den groben Überblick über das Projekt erläutert. In diesem Teil konzentriere ich mich auf die Kommunikation zwischen dem SAP Netweaver Gateway 2.0 und dem SAP R3.

Zwischen dem SAP Netweaver Gateway 2.0 und dem SAP R3 sollen bei diesem Projekt Fotos zu einem Transport geschickt werden. Anschließend soll im R3 das Foto als Anhang zum Transport gesichert werden (dieser Schritt wird aber hier nicht behandelt). Wie man vom SAP Netweaver Gateway 2.0 aufs SAP R3 zugreift habe ich bereits in einem [Beitrag](/sap-netweaver-gateway-2-0) geschrieben. Hier werde ich nur mehr auf die Details der Schnittstelle und Datenübertragung eingehen.

Als erstes muss ja die Transportnummer übergeben werden (zu welchem Beleg soll das Foto verknüpft werden) und anschließend noch die Binärdaten (bzw. gewünschte Dateiname) des Fotos. Ich habe mir die Option offen gelassen, weitere Parameter "frei" zu gestalten und nicht direkt in der Schnittstelle zu fixieren. Im Web-Umfeld hat sich für solche Zwecke bereits die JSON-Notation durchgesetzt (analog zu XML, das mir aber zu viel Overhead hatte). Wie man die JSON-Notation in SAP effektiv lesen kann, habe ich auch bereits in einem [Beitrag](/2012/05/json-datenformat-fur-sap) geschrieben. Ein Vorteil dieser Methode ist, dass ich bei späterer Änderung der Parameter die Schnittstelle vom RFC Baustein bzw. im SAP Netweaver Gateway 2.0 nichts ändern muss.

Somit steht also die Schnittstelle vom RFC Baustein fest und dieser kann implementiert werden:

```abap
FUNCTION zbc_file_upload_gateway.

*"-----------------------------------------------
*"Lokale Schnittstelle:
*" IMPORTING
*" VALUE(IV_FILENAME) TYPE FILEINTERN
*" VALUE(IV_FILECONTENT) TYPE STRING
*" VALUE(IV_DEBUG) TYPE FLAG DEFAULT SPACE
*" VALUE(IV_PARAMS) TYPE STRING OPTIONAL
*" VALUE(IV_LOGIC) TYPE ZBC_LOGIC
*"-----------------------------------------------

DATA: lv_content TYPE xstring.

DATA: ls_params TYPE zbc_key_value.

DATA: lt_params TYPE zbc_key_value_t.

DATA: lo_json_doc TYPE REF TO zcl_json_document.

IF iv_debug IS NOT INITIAL.
  DATA dummy TYPE c VALUE space.
  WHILE dummy IS INITIAL.
  ENDWHILE.
ENDIF.

" Parameters are JSON -> convert to internal Table

lo_json_doc = zcl_json_document=>create_with_json( iv_params ).

WHILE lo_json_doc->get_next( ) IS NOT INITIAL.
  CLEAR: ls_params.
  ls_params-key = lo_json_doc->get_value( 'key' ).
  ls_params-value = lo_json_doc->get_value( 'value' ).
  APPEND ls_params TO lt_params.
ENDWHILE.

" decode BASE64 coded file
CALL FUNCTION 'SSFC_BASE64_DECODE'
  EXPORTING
    b64data = iv_filecontent
  IMPORTING
    bindata = lv_content
  EXCEPTIONS
    OTHERS = 8.

" from here you can work with the file -> i.e. put it as an attachment to a transport
CASE iv_logic.
  WHEN 'TRPH'.
  " put file as an attachment to a transport

  WHEN OTHERS.
ENDCASE.

ENDFUNCTION.
```

Der aufmerksame Leser wird drei weitere Punkte bei dem Baustein gefunden haben:

* der Parameter _iv_debug_ (inkl. Endlosschleife)
* der Parameter _iv_logic_
* der Funktionsbaustein _SSFC_BASE64_DECODE _

Den Parameter _iv_debug_ verwende ich gerne zum debuggen eines RFC Bausteins. Wird der Parameter gesetzt, bleibt das Programm in einer Endlosschleife hängen und kann über die Transaktion SM50 in den Debugger geholt werden. Dazu braucht man Änderungsberechtigungen beim Debuggen, da man sonst die Variable _dummy_ nicht ändern kann und somit in der Schleife festhängt. Nur um die Parameter zu kontrollieren, reichen normale Debugging-Berechtigungen. Kleiner TIPP: in der Endlosschleife keinen Sleep einbauen, da das Programm sonst nicht in der SM50 aufscheint!

Der Parameter _iv_logic_ kann verwendet werden, um die tatsächliche Logik, was mit der hochgeladenen Datei zu tun ist, anzuspringen. Wie man am Namen des Funktionsbausteins sieht, kann dieser für jegliches Hochladen einer Datei verwendet werden; nur die Weiterverarbeitung der Datei (z.B. das Anhängen an einen Transportbeleg) hängt vom jeweiligen Prozess ab.

Der Funktionsbaustein _SSFC_BASE64_DECODE_ wird verwendet, um die Binärdaten (für die Übertragung im Web mit BASE64 codiert) des Fotos wieder in ihre binäre Form zu konvertieren.

Somit wäre die Arbeit auf dem SAP R3 System erledigt. Jetzt muss der Service (inkl. RFC-Aufruf im R3) noch am SAP Netweaver Gateway 2.0 eingerichtet werden. Wie man Grundsätzlich einen Service im SAP Netweaver Gateway 2.0 einrichtet, habe ich bereits in diesem [Beitrag](/2012/01/sap-netweaver-gateway-2-0) geschrieben.

Ich habe das Datenmodell _ZTRANSPORT_PHOTO_UPLOAD_ angelegt und dazu die Operation _CREATE_ mit folgendem Mapping zum RFC Baustein _ZBC_FILE_UPLOAD_GATEWAY_ erstellt:

![Gateway mapping](/images/2012/05/mapping_file_upload.png){: .align-center}

Anschließend habe ich noch ein Consumption Model zu diesem Datenmodell angelegt und somit kann der RFC-Baustein über den SAP Netweaver Gateway 2.0 von einem mobilen Endgerät aus aufgerufen werden.
