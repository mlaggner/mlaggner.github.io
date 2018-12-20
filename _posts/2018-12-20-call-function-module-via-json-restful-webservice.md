---
title: 'Call Function Module via JSON (Restful Webservice)'
categories:
  - programming
tags:  
  - sap
  - abap
summary:
  - image: sap.gif  
---

To connect to a SAP instance for sending/receiving data you have various options:

- Remote Function Call (RFC)
- SOAP Webservice
- OData Interface (via SAP Netweaver Gateway)
- Own implemented Webservice via SICF handler class
- ...

All of these options have advantages and disadvantages:

- RFC needs a binary library (librfc)
- SOAP Webservices have a huge overhead which I wanted to avoid
- OData was too slow for my use case
- I did not want to implement an own webservice handler class (focus on core aspects of the projects)

So there was no  easy to use/fast interface for my actual project available. After some research I've sumbled accross the project [JSON Adapter for ABAP Function Modules](https://github.com/cesar-sap/abap_fm_json) which is exactly what I need: a fast and easy to use interface to my Function Modules.

After I've impoerted the code (via [SAPLINK](https://wiki.scn.sap.com/wiki/display/ABAP/SAPlink)), enabled the service in [SICF](https://github.com/cesar-sap/abap_fm_json#create-icf-service), created the [authorization object](https://github.com/cesar-sap/abap_fm_json#abap-authorization) I was able to use the Function Module outside of the SAP server via HTTPS.

The project I am creating is written in Java and I used the great library [Retrofit](https://square.github.io/retrofit/) to consume the webservices. After just a few minutes I was able to consume the Function Module from our SAP server via Java **without** any overhead (librfc/SOAP generated classes).
