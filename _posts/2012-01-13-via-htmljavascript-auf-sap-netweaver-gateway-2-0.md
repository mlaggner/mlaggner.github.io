---
title: via HTML/Javascript auf SAP Netweaver Gateway 2.0
categories:
  - programming
tags:  
  - sap
  - html/javascript
summary:
  - image: sap.gif  
---
In meinem vorgegangenen Artikel "SAP Netweaver Gateway 2.0" habe ich meine ersten Versuche im SAP Netweaver Gateway 2.0 beschrieben. Der nächste Schritt war für mich, den Sevice am Netweaver Gateway durch eine HMTL/Javascript Anwendung zu konsumieren.

Dazu gibt es bei den Beispielen von SAP ein nettes [Howto](http://wiki.sdn.sap.com/wiki/download/attachments/250646828/JavaScript+-+How+to+Guide.pdf?version=1&modificationDate=1318263992587), welches bei mir aber nur bedingt funktionierte. Ziel des Versuchs war, eine HTML Seite zu erstellen, welche lokal im Browser lief (ohne dahinter liegenden Webserver) bzw. später via [Phonegap](http://phonegap.com/) von einem mobilen Endgerät aufgerufen werden soll. Diese HTML Seite soll via Ajax die Daten vom SAP Netweaver Gateway (durch diesen wiederum vom R3-Testsystem) ermitteln. Schon bei den ersten Versuchen stellte sich heraus, dass die Aufrufe vom Desktop PC zum Netweaver Gateway aufgrund von Cross Domain Requests nicht funktionierten. Am mobilen Endgerät wiederum funktionierten die Aufrufe - nur wie soll man eine HTML/Javascript Applikation entwickeln, wenn man diese nur am mobilen Endgerät aufrufen (und damit nicht debuggen) kann?

Die Lösung habe ich in einem [Beispielprogramm](https://github.com/phunkphorce/phonegap-netweaver-gateway) von Oscar Renalias gefunden. In diesem Beispielprogramm wird der Ajax Request über einen Proxy umgeleitet, falls der Client ein Desktop ist. Somit ergibt sich folgender Aufbau:

<figure><a href="/images/2012/01/aufruf_gw.png"><img src="/images/2012/01/aufruf_gw.png" alt="Aufruf Gateway"></a></figure>

Den Proxy habe ich ganz einfach über XAMPP/Apache gelöst, indem ich dort einen Reverse Proxy Eintrag für den SAP Netweaver Gateway erstellt habe:

```
#
# Reverse Proxy
#

ProxyRequests Off
ProxyPreserveHost On
ProxyPass /proxy/http://server:8042 http://server:8042
ProxyPassReverse /proxy/http://server:8042 http://server:8042
ProxyPass /proxy/http://server:8042 http://server:8042
ProxyPassReverse /proxy/http://server:8042 http:/server:8042
```

Daraufhin konnte ich mein Testprogramm fertig stellen. Beim Testprogramm habe ich direkt schon die Javascript Bibliotheken für eine Mobile Anwendung (Phonegap, JQueryMobile) integriert, da ich das ganze gleich auch im Android Emulator getestet habe. Das Programm besteht aus folgenden Komponenten:

* index.html

```
SAP Netweaver Gateway 2.0 Example

<script charset="utf-8&#8243; type="text/javascript" src="lib/phonegap-1.1.0.js"></script><script type="text/javascript" src="lib/jquery/jquery-1.6.4.js"></script>
<script type="text/javascript" src="lib/desktop.js"></script><script type="text/javascript" src="lib/jqm/jquery.mobile.js"></script>
<script type="text/javascript" src="lib/datajs-1.0.2.js"></script><script type="text/javascript" src="lib/kdauf.js"></script>
<script type="text/javascript" src="lib/script.js"></script>

<!- Einstiegsbild / Selektionsbild ->
</pre>
  <div id="main" data-role="page">
    <div data-role="header" data-position="inline">
      <h1>Kundenaufträge</h1>
      </div>
    <div data-role="content"></div>
  </div>
<pre>
```

* die Bibliotheken: Phonegap, JQuery, JQueryMobile und DataJS (benötigt für den Datentransport vom SAP Netweaver Gateway)
* desktop.js (für den Proxy zum Testen; wenn die Clientplattform Windows ist, dann route über den Proxy)

```
var isDesktop = ( navigator.platform.indexOf("Win") >= 0 );
String.prototype.prx = function() {
  if(isDesktop)
    return("http://localhost/proxy/" + this);
  else
  return(this);
}
```

* kdauf.js (das Handling der Kundenaufträge, die vom SAP Netweaver Gateway kommen)

```
KDAUF = function(data) {
  this.vbeln = data.value;
  this.kunnr = data.kunnr;
  this.vkorg = data.vkorg;
  this.erdat = data.erdat;
}

KDAUF.data = [];

KDAUF.getAll = function(callback) {
  var target = "http://server:8042/sap/opu/sdata/sap/GETVBAK/ztest_vbakCollection";
  var mandt = "?sap-client=001&sap-user=developer&sap-password=ch4ngeme";
  var filter = "&$filter=kunnr EQ '0000010021' AND vkorg EQ '0100'";
  var requestUri = target + mandt + filter;
  var kdaufs = [];

  OData.read( requestUri.prx(), function (data) {
    //Success Callback (received data is a Feed):
    console.log(data.results.length + " kdaufs loaded");
    for (var i = 0; i < data.results.length; i++) {
      var kdauf = new KDAUF({
        value: data.results[i].value,
        kunnr: data.results[i].kunnr,
        vkorg: data.results[i].vkorg,
        erdat: data.results[i].erdat,
      });
      kdaufs.push(kdauf);

      // keep it memory for later usage
      KDAUF.data[kdauf.vbeln] = kdauf;
    }

    callback(kdaufs);
  }, function(error) {
    alert("Error occurred " + error.message);
  });
  return(kdaufs);
}

KDAUF.getByVbeln = function(id) {
  return(KDAUF.data[id]);
}
```

* und script.js (triggern des Ladens)

```
$(document).ready(function() {
  $.mobile.page.prototype.options.addBackBtn = true;
  loadData();
  function loadData() {
    // show loading image
    $.mobile.showPageLoadingMsg();
    KDAUF.getAll(function(kdaufs) {
      var list = $('#kdauf-list');
      $.each(kdaufs, function(index, kdauf) {
        list.append('</pre>
                     <ul>
                     <li class="element">' + kdauf.vbeln + " " + kdauf.erdat + '</li>
                     </ul>
                     <pre>');
      });
      $("li")

      $.mobile.hidePageLoadingMsg();
      list.listview('refresh');
    });
  };
})
```

Das Testprogramm konnte daraufhin im Desktop-Browser aufgemacht werden (Vorsicht - in dem Fall muss die index.html via XAMPP geöffnet werden, sonst zählt der Proxyaufruf auf localhost auch als Cross Domain Request) und via Phonegap am Android Emulator:

<figure><a href="/images/2012/01/testprogramm_browser.png"><img src="/images/2012/01/testprogramm_browser.png" alt="das Testprogramm im Browser"></a></figure>
<figure><a href="/images/2012/01/testprogramm_android_emulator.png"><img src="/images/2012/01/testprogramm_android_emulator.png" alt="das Testprogramm im Android Emulator"></a></figure>

Damit steht jetzt nichts mehr im Wege, um eine größere HTML Anwendung zu schreiben, die mit SAP kommuniziert.
