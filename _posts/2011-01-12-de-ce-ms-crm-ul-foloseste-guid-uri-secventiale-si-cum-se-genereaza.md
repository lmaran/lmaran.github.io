---
layout: post
title: "De ce MS CRM-ul foloseste GUID-uri secventiale si cum se genereaza"
date: 2011-01-12 00:00:01
comments: true
categories: MS-SQL MS-CRM
---

Un exemplu de GUID-uri preluate din MS CRM:

![](/assets/images/2011/GUIDcrm.png)

In SQL, functia de baza care genereza GUID-uri este `NEWID()`. Problema la aceasta functie este ca numerele rezultate sunt intr-o forma aleatoare facand astfel indexarea complet neperformanta (indexarea se bazeaza pe algoritmul B-tree)

Incepand cu SQL2005 s-a introdus functia `NEWSEQUENTIALID()` care genereaza GUID-uri intr-o ordine secventiala. Daca se restarteaza serverul, secventialitatea (si unicitatea) numerelor generate se pastreaza dar baza de pornire va fi alta.
