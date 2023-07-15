---
layout: post
title: "SSRS 2008R2 - 'Unable to connect to the remote server'"
date: 2011-02-02 00:00:01
comments: true
categories: MSSQL-RS
---

Mesajul de eroare:

![](/assets/images/2011/SSRSerror.png)

**Rezolvare:** In fisierul `rsreportserver.config` modifica valoarea cheii `SecureConnectionLevel` din '2' in '0'.

![](/assets/images/2011/SSRSsol.png)

Pas 2: (path=C:\Program Files\Microsoft SQL Server\MSRS10_50.MSSQLSERVER\Reporting Services\ReportServer)

**Rezultatul:**

![](/assets/images/2011/SSRSok2.png)
