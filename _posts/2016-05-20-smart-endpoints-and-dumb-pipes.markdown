---
layout: post
comments: true
title:  "Smart end-points and dump pipes"
excerpt: "Are we doing integration wrong? Bulding business logic into interfaces and creating dependencies on bigger and bigger integration teams."
date:   2014-09-10 09:00:00
mathjax: false
---

<img src="/assets/smart-endpoints/sap_integration.png">

I've been working in the integration space with SAP products for many years now. In that time there’s always been the thirst for best practices not just from the SAP ecosystem but the wider software industry.
One of the more pervasive thoughts is __“smart end-points & dumb pipes”__. This is the notion that its your business systems that should be smart and not your interfaces. Its seen as an anti-pattern to load up your interface with complex business rules and decision making processes — effectively building another business application that will require never-ending support. Before you know it, you have a huge integration team looking after XI (_I like the sound of XI better than PI/PO -> SAP, please change it back!_).

SAP themselves used to preach the idea that you should not put business logic into your interfaces / integration tools — it belongs in your business systems. I remember this strongly from my first SAP XI course way back when version 3.0 first came out.
In the modern day, __micro-services__ are becoming all the rage… And again they follow that same philosophy of dumb pipes and smart end-points.

Another best practice has always been the use of asynchronous services for application-to-application integration, especially for updates. SAP also used to push this mantra strongly with the introduction of XI.
One of the selling points for XI was the guaranteed delivery mechanism which is enabled by asynchronous processing.

A while back now SAP also introduced ABAP proxies so that we could very easily build web-service communication between ERP and XI over HTTP. For new interfaces, the best way was to create an ABAP proxy. However, if an IDoc already existed that fit-the-purpose then you would re-use it. The other connector technology that was frowned upon (for all but lookups) was RFC.

So over the years consultants like myself have evangelised these best-practices with our customers…

### So… Why has SAP thrown all this out the door..?
It seems that these best-practices of old are now dead and buried at SAP. Their recent cloud products are a good example. New IDoc’s have been created and synchronous communication is the norm. What the..? Not only are new IDoc’s created, but its recommended that they be sent to XI via a soap channel — thereby making them synchronous. You have nothing to gain by routing these through XI. Why didn't they create an asynchronous ABAP proxy?

__Current documentation for integration of ERP with cloud products throws out the established rules and makes a mockery of us in the eye of SAP customers.__

### What should we be doing?
My point of view has not changed:

__Create ABAP proxies__ for new ERP interfaces if an existing IDoc does not suit the purpose (or if there exists a bloated, convoluted SAP Enterprise Service which is usually unlikely as they seem to have been built with a specific use-case in mind such as integration between ERP and SRM for example.)

__Use asynchronous processing__. _Firstly, everything is synchronous…_ But we abstract from this to provide an asynchronous layer and hence quality of service (e.g. Exactly-Once, Exactly-Once-In-Order).

Without this abstraction; synchronous processing is best-effort only. You may as well not use XI for synchronous integration as its not really offering you anything unless you need to translate to file/ftp or something else that may be a legacy format. With async you have “better” loose-coupling; non-blocking; re-send; guaranteed delivery (as SAP used to call it); exactly-once processing and so on.

__No business logic in your interfaces__. Mappings should be structural and value-related only. Avoid using BPM as much as technically possible. It creates a maintenance headache forever and you’re effectively building a new application to support as well as an interface.

### Its not easy to always follow these best practices
Sometimes the systems you are calling from XI can only do synchronous web services or are very chatty due to offering a horrible API, etc. So what do you do in these cases…

Well, the ideal solution is to wrap those applications that can’t talk properly in something that can. If your company has purchased an application you need to interface with SAP ERP and its application-to-application integration offering is horrible (like you need to call multiple web services to check if something exists, before updating it. And you need session keys, which are more services and so on and so on) then wrap it in a thin layer that does offer a good and proper asynchronous web service API, which can be easily connected to XI for hassle free integration. This wrapper application stays in the domain of the purchased app.

The wrapper could be built with some basic coding using any language that your company is fluent in and can talk via web services. Or it could be a simple API gateway like that from Axway as an example.

Your XI interfaces stay simple and easy to manage and support.

Alternatively, you may have to take advantage of a java mapping program in your XI interface to perform lookups, etc or get session keys.

_Worst case — you may have to use the dreaded BPM. But beware!_
