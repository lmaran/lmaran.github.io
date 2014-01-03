---
layout: post
title:  "C# Helper: RemainingTime() - pentru sortare implicita in Azure Table"
date:   2012-04-19 00:00:02
comments: true
categories: C# AzureTable
---

Am creat cateva metode simple care returneaza timpul scurs pana la o anumita data, in diverse unitati de masura: ticks, secunde, minute etc.

[Cod + Live Demo](https://compilify.net/qh/11) (comanda "run")

Un exemplu de rezultat este cel de mai jos:

![](https://dl.dropboxusercontent.com/u/43065769/blog/images/2012/RemainingTime.png)

## La ce sunt bune aceste cifre? ##

- ca prefix la PK/RK pentru sortare implicita in Azure Tables
- ca prefix la slug-ul unei resurse web (unicitate URL poza)
- atunci cand vrei sa generezi un ID mai “prietenos” decat un GUID
- etc

## De ce 2042? ##

Am considerat acest an ca fiind un bun compromis intre numarul de cifre necesare reprezentarii numerelor de mai sus si durata de viata maxima a unei aplicatii. Spre exemplu, daca "urc" la 2043, numarul de cifre creste cu o unitate pt. Ticks si Secunde. Daca vreau sa scad numarul de cifre cu o unitate, trebuie sa "cobor" anul la 2014 (sper ca aplicatia mea sa fie in functiune si dupa aceasta data -:)

[Puteti simula](https://compilify.net/qh/11) si alte variante, ruland codul cu diverse valori pentru anul final (yearEnd).