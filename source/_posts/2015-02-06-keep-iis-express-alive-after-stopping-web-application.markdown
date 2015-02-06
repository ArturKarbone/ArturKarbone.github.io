---
layout: post
title: "Keep IIS Express alive after stopping web application"
date: 2015-02-06 10:22:37 +0200
comments: true
categories: 
---



I faced a situation when IISExpress stops after I stop my debugging session. Not always convenient especially if I want to tweak/adjust some html/javascript code.
The good news is that Visual Studio allows us to configure the behaviour.

I've figured out at least two ways to do that by **unchecking** "Edit And Contitue" feature:

	1. Right click on your web project Properties->Web->Enable Edit And Continue 

{% img left /images/keep-iis-express-alive-after-stopping-web-application/IISExpress.png %}

	2. Go to Tools->Options->Debugging->Edit And Continue 

{% img left /images/keep-iis-express-alive-after-stopping-web-application/IISExpress2.png %}

