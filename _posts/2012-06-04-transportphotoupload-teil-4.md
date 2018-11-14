---
title: 'TransportPhotoUpload - Teil 4'
categories:
  - programming
tags:
  - html/javascript
  - sap
summary:
  - image: sap.gif
---
Im letzten Teil dieser Serie beschreibe ich, wie die Kommunikation zwischen dem Endgerät (HTML/Javascript) und dem SAP Netweaver Gateway 2.0 stattfindet.

Die Datenübertragung zwischen Javascript und dem SAP Netweaver Gateway 2.0 erfolgt mittels dem OData Protokoll. Eine Anleitung, wie man von Javascript mit dem OData Protokoll auf den SAP Netweaver Gateway 2.0 zugreifen kann, habe ich bereits in einem [Beitrag](/via-htmljavascript-auf-sap-netweaver-gateway-2-0) geschrieben. Auch das Problem mit dem Cross Domain Requests wird dort behandelt. Daher können wir uns direkt der Datenübertragung widmen.

Wie in den vorhergehenden Beiträgen beschrieben wurde, ist die Infrastruktur seitens SAP bereits eingerichtet und das HTML Programm soweit realisiert, dass der Barcode gescannt und die Fotos geschossen werden können. Jetzt müssen diese Fotos inkl. Metadaten ans SAP übertragen werden.

Als erstes Stand ich vor dem Problem, dass die Binärdaten der Fotos nicht direkt übertragen werden können. Daher müssen diese Binärdaten mit BASE64 kodiert werden und der BASE64 String kann anschließend übertragen werden. Um Binärdaten in Javascript auf BASE64 zu konvertieren fand ich recht schnell einen Algorithmus, der auch im SAP wieder erfolgreich dekodiert werden konnte. Leider stellte sich beim Testen heraus, dass der Typ _Uint8Array_, der im Algorithmus verwendet wurde, erst ab Android 3 verfügbar ist. Ein alternativer Algorithmus funktionierte auch nicht, daher habe ich das Problem anders gelöst: In der PhoneGap API fand ich die Filesystem API, mit der ich das File direkt schon BASE64 kodiert auslesen konnte. Daher habe ich am Desktop die Fotos über einen XMLHttpRequest + händisches Kodieren in BASE64 und am mobilen Endgerät über die Filesystem API gelöst:

```javascript
$.each( storageData, function(index, line) {
  if (line["type"] != "P")
    return;
  if(isDesktop){
    // on desktop via XMLHttpRequest
    var oXHR = new XMLHttpRequest();
    oXHR.open("GET", line["filename"], true);
    oXHR.overrideMimeType("text/plain; charset=x-user-defined");
    oXHR.responseType = "arraybuffer";
    oXHR.filename = line["filename"];
    oXHR.onload = function(oEvent) {
      var arrayBuffer = oXHR.response; // Note: not oXHR.responseText
      var content = base64ArrayBuffer(arrayBuffer);
      sendPicture(content, this.filename);
    };
    oXHR.send(null);
  } else {
    // on mobile via filesystem API
    window.resolveLocalFileSystemURI(line["filename"], function(fileEntry){
      fileEntry.file(function(file){
        var reader = new FileReader();
        reader.onloadend = function(evt) {
          var content = evt.target.result;
          // cut out base64 prefix
          var index = content.indexOf("base64,");
          if(index > 0) {
            index = index + 7;
            var length = content.length;
            content = content.substr(index, length - index);
            sendPicture(content, fileEntry.name);
          }
        };
        reader.readAsDataURL(file);
      }, function(evt) {
        console.log(evt.target.error.code);
      });
    }, function(evt){
      console.log(evt.target.error.code);
    });
  }
});
```

Somit kann am Desktop, sowie am mobilen Endgerät, das Foto in der Funktion _sendPicture_ erfolgreich versendet werden:

```javascript
/*
 * send picture
 */

function sendPicture(content, filename) {
  var target = "http://server:8042/sap/opu/sdata/sap/TRANSPORT_PHOTO_UPLOAD/ztransport_photo_uploadCollection";
  var mandt = "?sap-client=001&sap-user=developer&sap-password=ch4ngeme&!format=xml";
  var requestURL = target + mandt;
  var requestContent = {
    value : filename,
    debug : " ",
    content : content,
    params : '[{"key": "TRNUM", "value": "' + activeTransport + '"}]'
  };
  var request = {
    headers : {
      "X-Requested-With" : "XMLHttpRequest",
      "Content-Type" : "application/atom+xml",
      "DataServiceVersion" : "2.0"
    },
    requestUri : requestURL.prx(),
    method : "POST",
    data : requestContent
  };
  OData.request(request, function(data) {
    // Success Callback:
    alert("Photo successfully sent");
  }, function(err) {
    // Error Callback:
    alert("Error occurred " + err.message);
  });
}
```

Bei diesem Aufruf wird das Foto in BASE64 kodiert und die extra Parameter in JSON Notation übergeben. Über den SAP Netweaver Gateway 2.0 werden diese Parameter ans SAP R3 System weitergeleitet, wo diese dekodiert und verarbeitet werden (siehe frühere Beiträge).

An dieser Stelle sind die Grundzüge dieses Programmes fertig, und es bedarf nur mehr etwas Feinschliff.

Zusammenfassend biete ich noch eine Übersicht über die Probleme, über diese ich bei dem Programm gestolpert bin:

* Cross Domain Requests: Vor allem bei der Entwicklung und dem Testen am Browser war das ein massives Problem. Abhilfe schafft ein Proxy am PC (siehe diesen [Beitrag](/via-htmljavascript-auf-sap-netweaver-gateway-2-0))
* BASE64 kodieren von Binärdaten:  wenn die Binärdaten über einen XMLHttpRequest geladen und von diesem binären Format konvertiert werden, dann ist der Typ _Uint8Array_ nötig. Dieser Typ ist jedoch erst ab Android 3 verfügbar - also für Android Handys nicht verfügbar.
* Android:
  * _Uint8Array_ bei Android 2.x Handys nicht verfügbar
  * das Handy sollte mind. 5MP & Autofokus haben (sonst gibt es Probleme beim Scannen)
* Blackberry:
  * Deployment auf Blackberry war sehr sehr träge (ca 3 - 5 Minuten pro Versuch!)
  * BarcodeScanner Bibliothek war umständlich einzubinden (ca 1 Tag lang versucht die richtige Kombination aus Bibliothek & PhoneGap Version zu finden)
  * das Programm reagiert träge am Blackberry Torch 9800
  * das Blackberry Torch 9800 hat Speicherprobleme beim Übertragen der Fotos
  * die Photoqualität lässt sich nicht einstellen (daher 1MB große Fotos)
  * Logs auslesen via dem Blackberry ist umständlich
