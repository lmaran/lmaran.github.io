---
layout: post
title: "BackInfo.exe - afiseaza text dinamic pe desktop"
date: 2011-11-16 00:00:01
comments: true
categories: IT-tools
---

Presupun ca administrezi un numar mai mare de servere pe care le accesezi prin Terminal Services. Daca operezi pe mai multe masini simultan, atunci iti va fi de folos ca sa afli, dintr-o privire, pe ce masina te afli.

**BackInfo.exe** este un utilitar care iti permite sa afisezi, pe desktop, informatiile pe care le consideri relevante pentru o anumita masina. In cazul meu, pe langa numele server-ului si adresa de IP, am vrut "sa-mi sara in ochi" daca masina pe care lucrez este una de Productie, una de Test sau una de Dezvoltare. In final am configurat 3 template-uri care genereaza urmatoarele mesaje pe desktop:

![](/assets/images/2011/image6.png)

![](/assets/images/2011/image7.png)

![](/assets/images/2011/image8.png)

## Detalii despre instalare si configurare:

1. Descarca fisierul “.exe”. Detalii [aici](http://blogs.technet.com/b/johnbaker/archive/2006/02/15/where-can-i-find-the-backinfo-utility.aspx).
2. Pune intr-un folder fisierele `backinfo.exe` si `backinfo.ini`
3. Adauga in Startup, pe profilul "All Users", un shortcut care sa lanseze fisierul backinfo.exe. ("_C:\Documents and Settings\All Users\Start Menu\Programs\Startup_")

## Cum functioneza:

La fiecare operatie de logon se executa fisierul **backinfo.exe** care, pe baza informatiilor din `backinfo.ini` creaza imaginea `backinfo.bmp`. Aceasta imagine o depune in profilul utilizatorului curent, intr-un subfolder stabilit in fisierul .ini.

Informatia despre noul "bmp" se regaseste si in registry, la "_HK_Current_User/Control Panel/Desktop_"

```
key name:     Wallpaper
Data:         C:\DOCUME~1\lmaran\LOCALS~1\Temp\1\backinfo.bmp
```

## Setarile mele pentru backinfo.ini:

```
[General]
BackgroundColor = 0
AutoBackground = 1
UpdateDesktop = 1
ForceDesktopCenter = 1
Output = %temp%\backinfo.bmp
LineSpacing = 3
XOffset = 200
YOffset = -250

[Line1]
Font = Trebuchet MS
Size = 42
;Color = 16777215
Bold = 1
Italic = 0
Alignment = Center
ShadowX = 2
ShadowY = 2
ShadowColor = 4210752
Type = CompName


[Line2]
Font = Trebuchet MS
Size = 20
Color = 10526880
Bold = 0
Italic = 0
Alignment = Center
ShadowX = 0
ShadowY = 0
ShadowColor = 4210752
Type = NetInfo


[Line3]
Font = Trebuchet MS
Size = 30
Bold = 1
Italic = 0
Alignment = Center
ShadowX = 0
ShadowY = 0
;ShadowColor = 4210752
Type = FreeText

Test= ;(fara text)

;Text=Production Environment
;Color = 49152 ;(verde)

;Text=Test Environment
;Color = 24831 ;(portocaliu)

;Text=Development Environment
;Color = 16760832 ;(albastru)
```

Acesta este fisierul .ini pentru cazul default (fara informatia despre environment). Daca vrei sa adaugi si un mesaj personalizat, atunci poti modifica comentariile de la ultimele randuri sau poti pastra fisiere de configurare separate pentru fiecare caz in parte.

Succes!
