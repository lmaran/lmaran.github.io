---
layout: post
title:  "Azure File Service - performanta dezamagitoare"
date:   2014-06-15 00:00:01
comments: true
categories: software infrastructure
---

Ma refer la recent anuntatul serviciu de File Sharing, prezentat in detaliu [aici](http://blogs.msdn.com/b/windowsazurestorage/archive/2014/05/12/introducing-microsoft-azure-file-service.aspx).
In sine, serviciul este unul **foarte util**. L-am asteptat demult, fiindca intentionam sa-l folosesc ca un storage comun, in care sa-mi pastrez wwwroot-ul unui cluster format din doua VM. In felul acesta as fi scapat de sincronizarea datelor dintre cele doua masini.

Asadar, am reusit sa configurez IIS-ul sa-si ia datele din noua locatie:

 `\\myaccount.file.core.windows.net\myshare\inetpub\wwwroot\mysite`

Totul a functionat asa cum ma asteptam, mai putin performanta noului drive. Acesta s-a dovedit **cu un ordin de marime mai putin performant** decat celelalte doua disk-uri (C si D) disponibile "by default".

### Primele masuratori ###

Am inceput cu un test simplu - am copiat acelasi pachet de date, intre diferitele disk-uri ale masinii, cu urmatorul rezultat:

- Azure drive C to C --> 5 MB/sec
- Azure drive C to D --> 7.5 MB/sec
- Azure drive C to Z --> **0.4 MB/sec** (Z fiind mapat la noul **File Service**)

Eram curios cum se raporteaza aceste date la un disk SSD:

- homePC drive C to C --> 120 MB/sec

### Performantele Azure File Service: ###


Trecand acum intr-o zona putin mai riguroasa, am repetat masuratorile folosind doua dintre cele mai bine cotate benchmark-uri de profil: [ATTO](http://www.attotech.com/disk-benchmark/) si  [CrystalDiskMark](http://crystalmark.info/software/CrystalDiskMark/index-e.html).

![](https://dl.dropboxusercontent.com/u/43065769/blog/images/2014/azure-extrasmall-Z-drive-ATTO.png)

![](https://dl.dropboxusercontent.com/u/43065769/blog/images/2014/azure-extrasmall-Z-drive-Crystal.png)

### Rezultatele comparative: ###

![](https://dl.dropboxusercontent.com/u/43065769/blog/images/2014/azure-file-service-performance.png)

### Concluzie ###

In situatia in care, copierea unui kit de 30MB dureaza 15 secunde, pot spune ca serviciul are o aplicabilitate destul de redusa.

In momentul de fata am decis sa nu folosesc acest serviciu. Am de gand insa sa urmaresc indeaproape evolutia lui fiindca ma astept ca, atunci cand va trece de la faza de "Preview" la cea de "GA", produsul sa atinga acelasi standard de calitate ca si celelalte produse dezvoltate de Microsoft sub umbrela Azure. 