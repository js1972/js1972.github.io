---
layout: post
comments: true
title:  "Changing the default WD-ABAP Error Page"
excerpt: "I recently encountered a situation where I needed to change the standard WebDynpro Error Page to something that was seen to be a little more easy on the eye for users. The main requirement was for time-outs. It’s a pretty simple task except for a couple of gotcha’s that are not described in the SAP help..."
date:   2015-09-30 08:00:00
mathjax: false
---

I recently encountered a situation where I needed to change the standard web dynpro error page to something that was seen to be a little more easy on the eye for users. The main requirement was for time-outs. It’s a pretty simple task except for a couple of gotcha’s that are not described in the SAP help, which I’ll detail below.

The standard error page for a WD application can be changed in the ICF node. You can do this for an explicit application or for the entire WD node where your error page will be used for all sub-nodes (applications). *Note that it is not necessary to copy this change to all sub-nodes — the system will automatically use your error page on all sub-nodes in the ICF*.

<img src="/assets/wd-error-page/icf.jpg">

There are two main options for creating the error page:
1. “Explicit response time”
2. “Redirect to URL”

I’ll start with the second one first as I tried it out on my first attempt. Here you can insert any url which the user will be redirected to upon an application error (WD). This could be an error due to an internal short dump or a time-out.

I didn’t want to have to have a page sit on some other web server so started to write an error page in BSP. However when an error occurs, the user will be asked to logon. That’s not a good solution. It’s even worse if you’re using sso in the portal. This means I’d need to setup a service user in the ICF service for my bsp error page so that it would use the service users credentials. Again not ideal and I went back to the drawing board. (*also — you don’t seem to have access to any of the error variables described below when using this method.*)

The first option listed above, called “Explicit response time” allows you to directly insert some data for header values (which I admit I have no idea what they are for or what the benefit is — you can’t enter html here). It also lets you directly enter html for the **BODY** of the error page — this is what I used. I went about writing and testing the error page in eclipse and copied and pasted it into this body section.

<img src="/assets/wd-error-page/icf_edit.jpg">

<img src="/assets/wd-error-page/icf_html.jpg">

Unfortunately the formatting of your html is lost once you have saved — so keep your offline copy available for easy editing.

Okay — so far so good… This is where I started to run into troubles though. I needed to use some basic JavaScript to check the error type and display different messages if its from a time-out or not. I also needed to be able to handle some button clicks so the user could quickly and easily refresh the page or display further details for support.

No matter what I did the JavaScript just didn’t seem to get executed though. **The SAP Help — was no help, as is often the case!**

Luckily I came across note [#1342205](https://websmp130.sap-ag.de/sap/bc/bsp/spn/sapnotes/index2.htm?numm=1342205) which explained a few things. It seems that the onload event is not processed in the document due to the way that WD-ABAP uses ajax to insert the document into the DOM.

To get around this there is a trick that can be used. You need to use the onload event on a dummy IMG tag to get some script to execute on page load. This script is hidden within a span tag (in comments) as can be seen below:

<img src="/assets/wd-error-page/image_code.jpg">

Not the prettiest way to get some JavaScript to run on page load, but it works within the ICF error page framework. In the SPAN above you can see I do a basic check to see if the error message is a timeout or not and I change the displayed message based on this.

We have access to a suite of “**error variables**” (and some extra ones as shown here). You can access these on your page like this:

<img src="/assets/wd-error-page/error_variables.jpg">

I wrapped some of the variables above in DIV’s so I could access them in the JavaScript.

The end result looks like this:
<img src="/assets/wd-error-page/error_page.jpg">

<img src="/assets/wd-error-page/error_page2.jpg">

The Reload button simple causes a page refresh which takes the user back to where there were and the Show error details button just “shows” the DIV containing all the system variables shown in the code section above — which is of more use on an application error instead of the time-out shown here.

(Thanks goto [John Moy](http://scn.sap.com/people/john.moy3) for the ‘show more details’ idea.)

https://github.com/js1972/WebDynpro_ABAP_Error_Page

Originally published at [scn.sap.com](http://scn.sap.com/community/web-dynpro-abap/blog/2012/07/24/changing-the-default-wd-abap-error-page) on July 24, 2012.
