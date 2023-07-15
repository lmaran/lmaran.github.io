---
layout: post
title: "Activare 'WindowsAuthentication' pe IIS Express 8 (VS2012)"
date: 2012-09-06 00:00:01
comments: true
categories: Dev-tools, Windows-8
---

IIS Express 8 se instaleaza odata cu VS2012 sau separat.

## Problema:

In VS2012, daca creez un proiect de tip "Intranet Application" (MVC4 sau MVC3) si incerc imediat sa-l rulez, voi obtine mesajul "**Access is Denied**", cu detaliul:

"_Error message 401.2.: Unauthorized: Logon failed due to server configuration._"

![](/assets/images/2012/AccessDenied.png)

Ma conving ca problema tine de autentificare fiindca, daca anulez linia `<deny users="?" />` din web.config, atunci eroarea dispare. Din pacate, [aceasta](http://chanmingman.wordpress.com/2012/06/19/access-is-denied-in-mvc-4-intranet-application-in-vs-11/) nu poate fi considerata o solutie fiindca autentificarea devine astfel inutilizabila:

![](/assets/images/2012/NoAuth.png)

O solutie ar fi sa configurez proiectul a.i. sa fie rulat sub Cassini (IIS-ul integrat in VS2012). Am testat si merge OK.

O alta solutie (evident, mai buna) consta in activarea “Windows Authentication” in IIS Express. Pentru asta am editat fisierul "`applicationhost.config`" din "_\Users\lmaran.ETA2U\Documents\IISExpress\config_".

Concret, am modificat din "false" in “true” atributul din urmatoarea sectiune:

    <windowsAuthentication enabled="true">

## Rezultatul:

![](/assets/images/2012/Authok.png)

## Tips:

Fisierul de configurare din IIS express se poate accesa si direct, astfel:

![](/assets/images/2012/ShowIISExpress.png)

...selecteaza site-ul si click direct pe fisierul dorit.

![](/assets/images/2012/ShowIISExpresConfig.png)
