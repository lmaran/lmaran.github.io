---
layout: post
title:  "Ping - din Windows Azure"
date:   2014-06-15 00:00:01
comments: true
categories: software infrastructure
---

## Scenariu ##

Am auzit tot mai des in ultima vreme de [Digital Ocean](https://www.digitalocean.com/), un provider de infrastructura (IaaS) care iti ofera masini virtuale la performante si preturi tentante (ps: [vultr](https://www.vultr.com/) se vrea un DO killer). Nu am de gand sa renunt la cloud-ul Microsoft dar nici nu exclud posibilitatea ca, pentru anumite scopuri, sa-mi provizionez 1 sau 2 servere la DO pe care, ulterior, sa le accesez din Azure. Dar inainte de a decide o astfel de arhitectura, trebuia sa cunosc ce latenta exista intre datacenter-ele celor doi furnizori.

Eu folosesc datacenter-ul Microsoft din Amsterdam. Si Digital Ocean are un datacenter in Amsterdam iar acest lucru mi-a dat sperante ca pot obtine un timp de raspun bun intre cele doua retele. Asa ca am decis sa trec la masuratori, dar cum?

## Problema ##

ICMP-ul e dezactivat in Azure, prin urmare nici 'ping'-ul spre/dispre Azure nu functioneaza.

## Solutie ##

[PsPing](http://technet.microsoft.com/en-us/sysinternals/jj729731.aspx) - un command-line tool dezvoltat de Microsoft (mai precis, de bine cunoscutul [Mark Russinovich](http://en.wikipedia.org/wiki/Mark_Russinovich)) care iti permite sa trimiti pachete de control nu doar pe ICMP, ci si pe TCP.

## Mod de operare ##

Practic, utilitarul ofera 4 functionalitati de baza:

1. **ICMP ping**

 `psping -n 10 -w 4 speedtest-ams1.digitalocean.com`
 - 10 ping-uri (+4 de warmup)
 - asta nu merge din Azure
	
2. **TCP ping**
	
 `psping -n 10 -w 4 speedtest-ams1.digitalocean.com:80`
 - 10 ping-uri (+4 de warmup)
 - OBS: adaugi doar port-ul
 - cda de mai jos e lansata de pe o AzureVM hostata in Amsterdam (ca si Digital Ocean). De acasa, aceeasi comada -> 45ms
	
 ![](https://dl.dropboxusercontent.com/u/43065769/blog/images/2014/psping1.png)

3. **TCP latency test**
	
 `psping -l 8k -n 100 -w 4 speedtest-ams1.digitalocean.com:80`
 - request size = 8192 bytes
 - 100 request-uri (+4 de warmup)
 - OBS: daca pui un numar prea mare de request-uri (n) e posibil sa ti se inchida conexiunea inainte de finalizarea testului
 - cda de mai jos e lansata de pe o AzureVM hostata in Amsterdam (ca si Digital Ocean). De acasa, aceeasi comada -> 45ms (la fel ca in cazul 2)
 - daca la testul din fig. de mai jos (lansat din Azure) scad dimensiune pachetului de la 8k la 500 bytes -> obtin ac. rez. ca in cazul 2 (1.6ms)

 ![](https://dl.dropboxusercontent.com/u/43065769/blog/images/2014/psping2.png)
	
4. **TCP bandwidth test**
	
 `psping -b -l 8k -n 1000 -w 4 speedtest-ams1.digitalocean.com:80`
 - request size = 8192 bytes
 - 1000 request-uri (+4 de warmup)
 - cda de mai jos e lansata de pe o AzureVM hostata in Amsterdam (ca si Digital Ocean). 
  - de acasa, aceeasi comada ->1.2MB/s
  - de acasa, ac. cda dar inspre Azure -> 1.3MB/s
 Explicatia pt. latimea de banda asa de mica ar fi ca ambii provideri (DO si Microsoft) limiteaza traficul.

 ![](https://dl.dropboxusercontent.com/u/43065769/blog/images/2014/psping3.png)

## Observatii finale ##

Masuratorile le-am facut folosind portul 80 al masinii destinatie. Pentru ultimele 2 scenarii (latency si bandwidth test) utilitarul poate fi configurat si in mod 'server', caz in care asculta pe un anumit port la cererile primite de la client. Acest lucru presupune, evident, acces pe masina destinatie si deschiderea unui port in firewall.

 `psping -s mymachine:5000` 

## Concluzie ##

Folosind [PsPing](http://technet.microsoft.com/en-us/sysinternals/jj729731.aspx), am reusit sa descopar ca timpul de raspuns (latenta) dintre datacentere-le din Amsterdam ale Microsoft si Digital Ocean este una aproape neglijabila (2ms). Practic, ca si cand cele doua retele ar fi conectate in acelasi LAN. 
De aici incolo, orice scenariu devine posibil!

UPDATE: Am repetat testul Azure -> [Vultr](https://www.vultr.com/) (ambele in Amsterdam) iar timpul de raspuns a fost de 77ms. Prea mare!