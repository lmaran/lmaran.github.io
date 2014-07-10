---
layout: post
title:  "cURL in Windows"
date:   2014-06-23 00:00:01
comments: true
categories: software infrastructure
---

## Problema ##

Aparent nu ai nicio problema. Instalezi varianta de Windows (download [aici](http://curl.haxx.se/download.html#Win64) sau `cinst curl`) si totul functioneaza perfect:

![](https://dl.dropboxusercontent.com/u/43065769/blog/images/2014/curl1.png)

Problema apare atunci cand incerci sa rulezi comanda intre apostroafe (in loc de ghilimele). In acest caz vei primi eroarea:

`Protocol 'http not supported or disabled in libcurl`

![](https://dl.dropboxusercontent.com/u/43065769/blog/images/2014/curl2.png)

Dar de ce ai ai folosi apostroafe?...ar fi intrebarea fireasca. Pentru ca multe (majoritatea) din exemplele pe care le vezi pe net scrise in cURL provin din lumea Unix (ca si utilitarul, de altfel), iar aceste exemple sunt cu apostrof.

**Caz concret**: toate exemplele din documentatia [Elasticsearch](http://www.elasticsearch.org/guide/en/elasticsearch/reference/current/docs-index_.html) sunt de forma:

![](https://dl.dropboxusercontent.com/u/43065769/blog/images/2014/curl3.png)

Daca incerci sa rulezi comanda doar cu copy/paste, primesti eroarea de mai sus.

## Solutia 1 ##

- inlocuiesti apostroful cu ghilimele (`' --> "`) si
- prefixezi ghilimelele initiale cu un backslash (`" --> \"`)

Cu aceste modificari, comanda de mai sus ruleaza cu succes:

![](https://dl.dropboxusercontent.com/u/43065769/blog/images/2014/curl4.png)

Dar oare vrei sa faci asta pt. fiecare comanda pe care o experimentezi?

## Solutia 2 ##

Aceasta solutie presupune sa rulezi comanda `curl` intr-un 'shell' Unix. Sigur, tot in Windows.

O varianta ar fi sa instalezi [Cygwin](https://www.cygwin.com/) (`cinst cygwin`). Dezavantajul ar fi ca acest pachet poate fi mult prea mare (100MB) fata de ce ai nevoie.

O alta varianta (asta folosesc eu) ar fi sa te bazezi pe shell-ul `Git bash` pe care il ai deja instalat daca folosesti [Git for Windows](https://windows.github.com/). La mine l-am gasit in: `C:\Users\Lucian\AppData\Local\GitHub\PortableGit_054f2e797ebafd44a30203088cd3d58663c627ef`.

Il poti rula in doua moduri:

- `git-bash.bat` - lanseaza o consola proprie:

 ![](https://dl.dropboxusercontent.com/u/43065769/blog/images/2014/curl5.png)

- `bash.exe` (din subfolder-ul `bin`) - daca vrei sa ruleze in consola parinte:

 ![](https://dl.dropboxusercontent.com/u/43065769/blog/images/2014/curl6.png)

La final, adauga calea spre `git-bash.bat` sau `bash.exe` in variabila PATH.

## Concluzie ##

Multi producatori isi documenteaza REST API-urile folosind varianta cURL cu sintaxa Linux (ex: [Elasticsearch](http://www.elasticsearch.org/guide/en/elasticsearch/reference/current/docs-index_.html)). Inlocuid ghilimelele si apostroafele, aceste comnzi pot rula si pe varianta cURL de Windows. Daca vrei insa sa rulezi aceste comenzi fara nicio modificare (doar cu `copy/paste`), foloseste un shell de tipul `cygwin` sau `git-bash`.