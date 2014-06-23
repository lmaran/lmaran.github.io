---
layout: post
title:  "Windows Azure - ce DataCenter sa alegi?"
date:   2014-06-21 00:00:01
comments: true
categories: software infrastructure
---

## Criteriul de proximitate ##

Ca o regula simpla, timpul de raspuns este direct proportional cu distanta (geografica) dintre utilizatorii unei aplicatii si DataCenter-ul care gazduieste aplicatia.
Spre exemplu, daca aplicatia mea se adreseaza preponderent publicului din Romania, atunci - aplicand acest criteriu - voi alega DC-ul Azure din Amsterdam.

Dar hai sa vedem care sunt diferentele din cele 2 DC-uri disponibile in Europa. Eu cunosc **2 instrumente** de masura:

1. [http://azureping.info](http://azureping.info/)

 ![](https://dl.dropboxusercontent.com/u/43065769/blog/images/2014/azurepinginfo.png)

2. [http://azurespeedtest.azurewebsites.net](http://azurespeedtest.azurewebsites.net)

 ![](https://dl.dropboxusercontent.com/u/43065769/blog/images/2014/azure-speed-test-chart.png)

 ![](https://dl.dropboxusercontent.com/u/43065769/blog/images/2014/azure-speed-test-table.png)

Asadar, cel putin pentru Romania (ambele masuratori fiind facute din Tm) am putea folosi oricare din cele doua datacentere, cu performante  asemanatoare in ceea ce priveste timpul de raspuns.

## Interconectarea cu datacenter-ele altor furnizori ##

In caz ca trebuie sa decid intre 2 variante sensibil egale (vezi mai sus) va trebui sa aplic un 'criteriu de departajare'. Sa luam spre exemplu urmatorul scenariu:

Am o aplicatie care vreau sa ruleze pe o instanta cu hdd SSD de la [Digital Ocean](https://www.digitalocean.com/). In acelasi timp, vreau ca aplicatia sa foloseasca Azure Table ca baza de date nosql. 

Fiindca latenta dintre appLayer si dataLayer trebuie sa fie minima, trebuie ca datacenter-ele care gazduiesc cele 2 servicii sa fie cat mai aproape unul de altul. In final as alege:

 - Digital Ocean -> Amsterdam
 - Windows Azure -> Amsterdam (DO nu au datacenter in Dublin)

PS Am aratat [aici](http://maran.ro/2014/06/14/ping-din-windows-azure/) ca latenta dintre Digital Ocean si Azure este de doar 2ms.

## Concluzie ##

Desi ambele datacentere Azure din Europa ofera aceleasi performante, in cazul meu **am ales Amsterdam** datorita posibilitatii de a ma conecta, practic la viteza unui LAN, cu unul dintre furnizorii cu care intentionez sa 'colaborez': Digital Ocean.