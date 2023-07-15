---
layout: post
title: "VisualHg: o problema de instalare pe VS2012 si Windows 8"
date: 2012-09-30 00:00:01
comments: true
categories: Dev-tools
---

[VisualHg](http://visualhg.codeplex.com/) este un plugin pentru Visual Studio care iti permite sa manipulezi operatiile specifice versionarii (_commit_, _synchronize_, _repo browser_ etc) direct din mediul de dezvoltare.

Odata **TortoiseHg** instalat ( [am scris](http://maran.ro/2012/09/15/tortoisehg-mici-automatizari-personale/) despre asta cu putin timp in urma), pasul imadiat urmator ar fi instalarea lui **VisualHG**.

## Problema:

La finalul instalarii, plugin-ul nu apare ca fiind disponibil in VS2012 (la sectiunea _Tools_ – _Options_ – _Source Control_), chiar daca instalarea s-a facut fara probleme (adica fara vreun mesaj de eroare).

## Solutie:

Trebuia fortat VS2012 sa-si faca un "refresh" (un restart nu e suficient):

start Command Prompt cu drept de administrator:

    cd C:\Program Files (x86)\Microsoft Visual Studio 11.0\Common7\IDE\
    devenv /setup

Pot obtine apoi confirmarea ca instalarea s-a facut ok:

![](/assets/images/2012/VisualHgInstall.png)

Din acest moment, **VisualHG** ne permite sa apelam **TortoiseHG** direct din Visual Studio 2012.
