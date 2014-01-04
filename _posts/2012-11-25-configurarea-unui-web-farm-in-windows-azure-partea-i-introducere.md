---
layout: post
title:  "Configurarea unui Web Farm in Windows Azure (partea I - Introducere)"
date:   2012-11-25 00:00:01
comments: true
categories: Dev-infrastructure Azure
---

- partea I – Introducere
- partea II – [Pregatirea infrastructurii](http://lucian.maran.ro/2012/11/25/configurarea-unui-web-farm-in-windows-azure-partea-ii-pregatirea-infrastructurii/)
- partea III – [Instalarea primei masini virtuale (VM1)](http://lucian.maran.ro/2012/11/25/configurarea-unui-web-farm-in-windows-azure-partea-iii-instalarea-primei-vm/)
- partea IV – [Instalarea unei masini virtuale secundare (VM2)](http://lucian.maran.ro/2012/11/25/configurarea-unui-web-farm-in-windows-azure-partea-iv-instalarea-unei-masini-virtuale-secundare/)
- partea V – [Publicarea cu WebDeploy din VisualStudio 2012](http://lucian.maran.ro/2012/11/26/configurarea-unui-web-farm-in-windows-azure-partea-v-publicarea-cu-webdeploy-din-visualstudio-2012/)
- partea VI – [Sincronizare automata cu WebDeploy si PowerShell](http://lucian.maran.ro/2012/11/26/configurarea-unui-web-farm-in-windows-azure-partea-vi-sincronizarea-automata-cu-webdeploy-si-powershell/)

----------

## Ce este un Web Farm? ##

Doua sau mai multe servere capabile sa lucreze sincron pentru a deservi aceeasi aplicatie web.

Cea mai banala aplicatie care ar putea evidentia faptul ca ruleaza intr-un WebFarm (in cazul ASP.NET MVC) ar include un View care sa afiseze numele masinii pe care se executa codul:

    <div>HostName: @Environment.MachineName</div>

![](https://dl.dropboxusercontent.com/u/43065769/blog/images/2012/LoadBalanceDemo2.png)

## De ce un Web Farm? ##

Nu dau definitii si nu teoretizez, spun doar ce m-a interest pe mine:

- Disponibilitate (High Availability, ca sa fiu pretentios);
- Performanta, in perspectiva;

Sa detaliez cele doua aspecte:

- Vreau ca aplicatiile pe care le fac (in special servicii WebAPI) sa fie disponibile non-stop. Mai exact, daca Microsoft ofera un SLA de **99,95%** pt. doua  Web/Worker Rol-uri, consider ca si eu, cu minim doua VM-uri din Azure pot ajunge la aceasta valoare. [Pingdom](http://pingdom.com/) imi permite sa verific daca [sunt in graphic](http://api.idgenerator.net/) cu acest obiectiv. Si fiindca monitorizarea am configurat-o la cea mai "dura" valoare (1 min.), vreau sa evit si cele mai mici intreruperi, care ar putea fi provocate de:
	- restart-uri cerute de instalarea unui ServicePack sau a unei component software;
	- reporniri necesare in cazul unui upgrade/downgrade a instantei (Ex: ExtraSmall -> Small);
	- recrearea de catre Fabric Controller a masinii virtuale in urma unei defectiuni hardware;
- Daca aplicatia este la inceput si cu un "success" limitat (numar rezonabil de cereri de acces), un singur server ar oferi practic acceasi performanta ca si o ferma de doua servere cu o configuratie "la jumatate". Pe masura ce aplicatia creste, acum, in Azure, poti recurge la o **scalare pe verticala** de pana la 8 CPU si 16 GB RAM. Problema ar fi ca, de aici incolo, singura solutie ar fi adaugarea unui nou server la infrastructura actuala (**scalare pe orizontala**). Si, daca acest lucru este relativ simplu de facut d.p.d.v hardware (vom vedea asta in aceasta serie), nu acelasi lucru se poate spune si despre aplicatii. Acestea trebuie din start (in perspectiva) proiectate pentru a putea rula pe o infrastructura distribuita (ex: nu poti presupune ca 2 cereri consecutive vor fi tratate de acelasi server, deci nu te poti baza pe conceptual de *sesiune*).

## De ce Azure?  ##
(...cand mai toti providerii ofera masini virtuale)

- platesc cat consum (majoritatea ofera un abonament fix pe luna);
- am un portal prin care pot sa-mi provizionez singur cate masini doresc, inclusive upgrade/downgrade la configuratie;
- pot sa stabilesc destul de precis balanta dintre siguranta datelor, performanta si cost (Region, AffinityGroup, AvailabilitySet, GeoReplication);
- siguranta “by default”: storage cu tripla redundanta si restore automat, monitorizarea masinilor si failover automat;
- notorietate (de ce nu?): acum 8 datacenters pe 3 continente;

## Azure Virtual Machine: ##
Cateva precizari, poate mai putin cunoscute:

- VM are un IP privat iar accesul la orice VM se poate face doar prin intermediul unei masini intermediare (Load Balancer) care furnizeaza IP-ul **public** (VIP);
- Load Balancer-ul este gratuit si ofera un acces limitat dar sufficient pentru a configura o **Ferma de Servere**;
- IP-ul **public** ramane neschimbat atata timp cat nu stergi instanta.  Asta pentru ca acest IP se aloca doar in faza de deploy a masinii, nu si la restart/upgrade;
- Spre deosebire de un Web/Worker Role, la o VM nu pierzi informatia de pe disk in cazul unui failover. Asta pentru ca imaginea (**vhd**-ul) care sta la baza instantei este memorata intr-un blob personal, declarant in faza de instalare.

## Terminologie: ##
Doar notiunile care le consider mai putin cunoscute:

- **Regions** - toate serviciile configurate cu aceeasi Reg**iune vor rula in acelasi *Datacenter*.
- **Affinity Group** - toate serviciile configurate cu acelasi AG vor rula 'cat mai aproape' unul de celalalt.  Un *datacenter* are suprafata catorva stadioane si conteaza distanta dintre noduri sau numarul de 'hop'-uri. Concret, Fabric Controller-ul va stoca resursele in acelasi cluster din acelasi container (detalii [aici](http://social.technet.microsoft.com/wiki/contents/articles/7916.importance-of-windows-azure-affinity-groups.aspx)).
- **Availability Set** - Pe de o parte, pentru a creste performanta vrei ca resursele (sa zicem cele 2 masini dintr-un Farm) sa ruleze 'cat mai apropiat'. Primele doua optiuni te ajuta in acest sens. Dar ce se intampla daca resursele nimeresc in acelasi rack sau comunica prin acelasi switch? Aici intervine *AvailabilitySet* si iti garanteaza ca, doua resurse configurate cu acelasi AS nu vor ajunge niciodata in situatia de "*single point of failure*".
- **Geo-Replication** - Datele stocate in orice *Storage Acount* sunt memorate, transparent si **gratuit**, cu tripla redundanta, in acelasi datacenter. Siguranta oferita in acest caz este una mai mult decat decenta, dar nu te protejeaza si in faza unui calamitati. Pentru astfel de situatii, in Azure ai posibilitatea sa declari, la nivel de *Storage Account*, ce date doresti sa fie replicate la un alt *datacenter*, de pe acelasi continent (de data aceasta, cu un **cost** aferent pentru stocare si tranzactii).

Urmareste, in [partea a II-a](http://lucian.maran.ro/2012/11/25/configurarea-unui-web-farm-in-windows-azure-partea-ii-pregatirea-infrastructurii/), pregatirea infrastructurii pentru un WebFarm cu doua servere!