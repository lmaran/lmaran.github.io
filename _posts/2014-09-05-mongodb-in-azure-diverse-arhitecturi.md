---
layout: post
title: "MongoDB in Azure - diverse arhitecturi"
date: 2014-09-05 00:00:01
comments: true
categories: MongoDB, Azure
---

Am sa prezint cateva modalitati de interconectare, cu avantaje si dezavantaje, pentru un scenariu simplu: doua VM (un WebServer si un DataServer) hostate in Azure. Pentru exemplificare am ales un IIS/Windows si un MongoDB/Linux, dar aceste detalii sunt prea putin relevante.

Asadar, WebServer-ul il expui (cel putin portul 80). Dar DataServer-ul?

## Arhitectura #1

![](/assets/images/2014/09-05-mongo-azure-1.png)

**Avantaje:**

- server-ul de Mongo nu este expus in exterior - varianta cea mai sigura
- latenta dintre WebSrv si DataSrv este minima (~ 1-2 ms), comunicarea facandu-se pe un [Virtual Network](http://azure.microsoft.com/en-us/services/virtual-network/) intern, cu IP-uri private ([si statice](http://msdn.microsoft.com/en-us/library/azure/dn630228.aspx), daca doresti)

**Dezavantaje:**

- orice operatie de administrare (pe SO sau Mongo) trebuie sa o faci conectandu-te in prealabil pe o masina intermediara (Remote Desktop)
- in Azure, o VM poate fi accesata doar de catre masinai configurate sa ruleze in acelasi Virtual Network. Cum doar VM, WebRole si WorkerRole opereaza cu conceptul de VNet, aceasta solutie nu mai functioneaza in cazul in care decidem sa reutilizam acelasi server de MongoDB, apelandu-l din alte aplicatii ce ruleaza ca si Azure Web Sites - [deocamdata](http://feedback.azure.com/forums/169385-websites/suggestions/4135102-allow-azure-web-sites-to-connect-to-a-virtual-netw).

## Arhitectura #2

![](/assets/images/2014/09-05-mongo-azure-2.png)

**Avantaje:**

- spre deosebire de cazul precedent, conectarea la SO sau DB o poti face direct, fara sa folosesti o masina intermediara
- spre deosebire de cazul precedent, conectarea la MongoDB o poti face (si) dintr-un Azure Web Sites, prin acelasi endpoint public ca din orice locatie externa.
  - OBS: folosind acelasi WebSrv dar conectandu-ma la DataSrv via endpoint-ul public, am observat o crestere (dar nu semnificativa) a latententei, in plaja (~ 1-7 ms)
- poti pastra si comunicarea pe VNet-ul privat, caz in care latenta dintre WebSrv si DataSrv ramane minima (~ 1-2 ms)

**Dezavantaje:**

- server-ul de baze de date (MongoDB) este expus in exterior. Cu toate acestea, riscul unui atac reusit este practic inexistent daca iti iei toate masurile de protectie:

  - endpoint-urile de pe Azure LB (porturile publice) le configurezi cu alte valori decat cele standard (pt. Mongo sau SSH)
  - configurezi in ACL (portal Azure) permisiuni de acces doar pentru cererile care sosesc de pe masini cu adrese IP cunoscute (ex: PC-ul local sau chiar servere de productie)
  - activezi autentificarea pe baza de usr/psw in MongoDB (implicit este dezactivata)
  - SSL, etc, etc

## Arhitectura #3

![](/assets/images/2014/09-05-mongo-azure-3.png)

**Avantaje/Dezavantaje**

- dupa cum se observa, este o arhitectura identica cu cea precedenta, dar fara Virtual Network. Comunicare cu MongoDB se poate face exclusiv pe endpoint-ul public, fapt ce o transforma intr-o solutie mai putin flexibila.
- are dezavantajul ca WebSrv este "obligat" sa foloseasca o ruta ocolitoare si, dupa cum spuneam anterior, putin mai lenta.

## Concluzie

In cazul in care doresti sa folosesti aceeasi baza de date (acelasi _ConnectionString_) indiferent de environment (dezvoltare, test, prod) - alege optiunea #3.
Pentru toate celelalte cazuri, recomand arhitectura #2.
