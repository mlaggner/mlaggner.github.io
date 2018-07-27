---
title: JSON Datenformat in SAP aufbereiten
categories:
  - programming
tags:
  - sap
  - abap
summary:
  - image: sap.gif
---
Im Zusammenhang mit dem SAP Netweaver Gateway 2.0 musste ich eine Möglichkeit finden, eine dynamische Anzahl an Parametern einem RFC-Baustein auf unserem R3 System zu übergeben. Im Web-Umfeld wird so ein Problem oft mittels JSON-Notation gelöst und nach kurzer Suche fand ich auch eine Möglichkeit, wie ich das in SAP lösen konnte.

Es gibt ein Projekt namens [zJSON](https://cw.sdn.sap.com/cw/groups/zjson), welches für die Konvertierung von JSON in ABAP verwendet wird. Dieses Projekt kann über [SAPlink](http://code.google.com/p/saplink/) auf ein kompatibles SAP-System eingespielt werden.

Wurde das Projekt zJSON in das SAP-System (R3 in meinem Falle) eingespielt, kann man mit der Klasse _ZCL_JSON_DOCUMENT_ zwischen JSON und ABAP konvertieren. Bei meiner Applikation wird dem RFC-Baustein ein String mit Parametern in JSON-Notation übergeben und dieser sollte in die entsprechenden ABAP-Variablen konvertiert werden. Die erfolgt mit folgendem Code:

```
DATA: ls_params TYPE zbc_key_value.

DATA: lt_params TYPE zbc_key_value_t.

DATA: lo_json_doc TYPE REF TO zcl_json_document.

" Parameters are JSON -> convert to internal Table
lo_json_doc = zcl_json_document=>create_with_json( iv_params ).

WHILE lo_json_doc->get_next( ) IS NOT INITIAL.
  CLEAR: ls_params.

  ls_params-key = lo_json_doc->get_value( 'key' ).
  ls_params-value = lo_json_doc->get_value( 'value' ).
  APPEND ls_params TO lt_params.

ENDWHILE.
```

In diesem Coding werden alle, durch JSON übergebenen, Parameter in einer internen Tabelle (_lt_params_) gespeichert, damit später der Zugriff darauf komfortabler wird. Der Typ _zbc_key_value_ enthält nur 2 Felder (_key_ und _value_) welche vom Typ String sind.

Somit ist es mir jetzt möglich, über HTML Parameter in JSON-Notation an den SAP Netweaver Gateway 2.0 zu schicken, welcher diese wiederum über RFC an das R3 System weiterleitet.
