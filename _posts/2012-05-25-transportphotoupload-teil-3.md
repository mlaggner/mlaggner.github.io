---
title: 'TransportPhotoUpload - Teil 3'
categories:
  - programming
tags:  
  - android
  - html/javascript
  - sap
  - abap
summary:
  - image: sap.gif
---
In den ersten beiden Teilen habe ich einen Überblick über das Projekt und den Aufbau auf der SAP-Seite gegeben. Jetzt geht es an die Gestaltung der HTML5 Applikation bzw. Probleme, die bei der Umsetzung aufgetreten sind.

Als ersten Schritt habe ich ein PhoneGap-Projekt für Android angelegt, wie es auf der Homepage von PhoneGap [beschrieben](http://docs.phonegap.com/en/1.7.0/guide_getting-started_android_index.md.html#Getting%20Started%20with%20Android) ist. Nach einiger Recherche, welche UI-Bibliothek ich verwenden will, bin ich bei einer Kombination aus [jQuery](http://jquery.com/) und [jQuery mobile](http://jquerymobile.com/) hängen geblieben. Diese zwei Frameworks ermöglichten mir, eine optisch gut aussehende Oberfläche mit minimalen Aufwand umzusetzen. Zusätzlich funktioniert das Testen der Applikation mit dem Chrome Browser (bzw. Firefox + firebug extension) sehr gut.

Jetzt, nachdem die Grundzüge der Oberfläche erstellt wurden, kam es zur ersten Herausforderung: der Barcode auf dem Verladeschein muss gescannt werden und die Transportnummer wieder an die Applikation zurückgegeben werden. Gott sei dank gibt es für PhoneGap ein eigenes [Plugin](https://github.com/phonegap/phonegap-plugins/tree/master/Android/BarcodeScanner) zum Scannen von Barcodes. Diesen musste man zum einen in das PhoneGap Projekt integrieren (eigene Library) und die zusätzliche JavaScript Datei in der Webanwendung einfügen. Somit konnte das Plugin aus der HTML5 Applikation heraus aufgerufen werden:

```
/*
* scan barcode
*/

app.scan = function() {
  window.plugins.barcodeScanner.scan(function(result) {
    console.log("We got a barcode: " + result.text);
    app.navigateToTransport(result.text);
  }, function(error) {
    alert("Scanning failed: " + error);
  });
};
```

Das nächste Problem kam aber gleich im Anschluß: am Desktop im Browser hatte man natürlich keinen Zugriff auf eine Kamera und somit kann man vom Barcodescanner keine Transportnummer bekommen. Dazu habe ich mir eine kleine Hilfe geschrieben: mittels einer Funktion prüfe ich ab, ob ich von einem Desktop aus das Programm ausführe und generiere in diesem Fall eine fiktive Nummer:

```
var isDesktop = ( navigator.platform.indexOf("Win") >= 0 );

/*
* scan barcode
*/

app.scan = function() {
  if (isDesktop) {
    app.navigateToTransport(Math.floor(Math.random() * 100000001));
  } else {
    window.plugins.barcodeScanner.scan(function(result) {
      console.log("We got a barcode: " + result.text);
      app.navigateToTransport(result.text);
    }, function(error) {
      alert("Scanning failed: " + error);
    });
  }
};
```

Die Hilfe über _isDesktop_ half mir bei dem Projekt noch sehr oft, da man des öfteren am Desktop anders reagieren muss, als beim mobilen Gerät.

Da jetzt die Transportnummer richtig ermittelt wird, müssen die Fotos von der Kamera (bzw. Dateien am Desktop) geschossen werden:

```
var photoCount = 1;

/*
* take a picture
*/

app.takePicture = function() {
  if (isDesktop) {
    onTakePictureSuccess("test" + photoCount + ".jpg");
    photoCount += 1;
  } else {
    navigator.camera.getPicture(onTakePictureSuccess, onTakePictureFail, {
      quality : 50,
      destinationType : Camera.DestinationType.FILE_URI
    });
  }
};
```

Jetzt können nach erfolgreichem Anscannen des Verladescheins Fotos zu einem Transport geschossen werden. Bis jetzt läuft das Ganze aber nur einmal - man kann diesen Prozess noch nicht unterbrechen und nachträglich Fotos schießen. Dazu habe ich die Informationen zu angescannten Transporten und geschossenen Fotos in der lokalen Datenbank [jStorage](http://www.jstorage.info/) gespeichert. Da in dieser Datenbank nur Key/Value Paaere gespeichert werden können, musste ich zu einem Trick greifen, damit etwas komplexere Daten in dieser Datenbank gespeichert werden können. Sehen wir uns das am Beispiel des Schrittes nach dem Anscannen eines Barcodes an: hier werden alle bereits gespeicherten Fotos zu diesem Transport ausgelesen und dargestellt.

```
/*
* navigate to detail screen
*/

app.navigateToTransport = function(transport) {
  if (transport > "") {
    // get all saved fotos for this transport
    storageData = $.jStorage.get(transport, new Array());
    if (storageData.length == 0) {
      // new transport
      storageData.push({
        type : "M",
        timestamp : new Date().getTime()
      });
    }
    // page content
    var list = $('#photoList');
    $.each(storageData, function(index, line) {
      if (line["type"] != "P")
        return;
      var id = line["id"];
      var filename = line["filename"];
      list.append('<li class="element" id="' + id + '"><a data-photoId="' + id + '" href="#photo?id=' + id + '">'
      + filename + '</a><a href="#deletePhoto?id=' + id
      + '" data-rel="dialog" data-transition="pop">Delete Photo</a></li>');

      photoCount += 1;
    });
  }
};
```

Und hier noch am Beispiel vom Foto schießen:

```
/*
* picture sucessfully taken
*/

function onTakePictureSuccess(imageURI) {
  var date = new Date();
  var id = crc32(imageURI + date.getDate() + date.getTime());
  // save the url of this picture in storage
  storageData.push({
    type : "P",
    id : id,
    filename : imageURI
  });

  jQuery.jStorage.set(activeTransport, storageData);

  // add this picture to the list of pictures
  var list = $('#photoList');
  list.append('<li class="element" id="' + id + '"><a href="#photo?id=' + id + '">' + imageURI
  + '</a><a href="#deletePhoto?id=' + id + '" data-rel="dialog" data-transition="pop">Delete Photo</a></li>');
  $("li")

  $.mobile.hidePageLoadingMsg();
  list.listview('refresh');
}
```

Auf diese Weise hat kann man auch eine Historie erstellen, welche Transporte in letzter Zeit angescannt wurden und welche Fotos zu diesen Transporten bereits geschossen wurden. Das waren jetzt grob die Schritte, wie man das Scannen des Barcodes und das Schießen der Fotos (inkl. Historie) über PhoneGap realisiert.

Anbei noch 2 Screenshots, wie diese Applikation im Browser aussieht:

<figure><a href="/images/2012/05/gui_1.png"><img src="/images/2012/05/gui_1.png" alt="Einstiegsbild mit Historie"></a></figure>
<figure><a href="/images/2012/05/gui_2.png"><img src="/images/2012/05/gui_2.png" alt="einzelner Transport mit Fotos"></a></figure>

Diese Beispiele sind nur Auszüge (das ganze Coding wäre zu lang), ich kann aber gerne bei Fragen weiterhelfen. Bitte mich dazu einfach kontaktieren oder kommentieren.

Im nächsten und letzten Teil widme ich mich der Kommunikation zwischen dem mobilen Gerät und dem SAP Netweaver Gateway 2.0.
