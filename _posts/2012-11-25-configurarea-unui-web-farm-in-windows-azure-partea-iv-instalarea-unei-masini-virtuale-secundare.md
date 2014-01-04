---
layout: post
title:  "Configurarea unui Web Farm in Windows Azure (partea IV - Instalarea unei masini virtuale secundare)"
date:   2012-11-25 00:00:04
comments: true
categories: Dev-infrastructure Azure
---

- partea I – [Introducere](http://lucian.maran.ro/2012/11/25/configurarea-unui-web-farm-in-windows-azure-partea-i-introducere/)
- partea II – [Pregatirea infrastructurii](http://lucian.maran.ro/2012/11/25/configurarea-unui-web-farm-in-windows-azure-partea-ii-pregatirea-infrastructurii/)
- partea III – [Instalarea primei masini virtuale (VM1)](http://lucian.maran.ro/2012/11/25/configurarea-unui-web-farm-in-windows-azure-partea-iii-instalarea-primei-vm/)
- partea IV – Instalarea unei masini virtuale secundare (VM2)
- partea V – [Publicarea cu WebDeploy din VisualStudio 2012](http://lucian.maran.ro/2012/11/26/configurarea-unui-web-farm-in-windows-azure-partea-v-publicarea-cu-webdeploy-din-visualstudio-2012/)
- partea VI – [Sincronizare automata cu WebDeploy si PowerShell](http://lucian.maran.ro/2012/11/26/configurarea-unui-web-farm-in-windows-azure-partea-vi-sincronizarea-automata-cu-webdeploy-si-powershell/)

----------

Daca WebFarm-ul are mai mult de 2 masini virtuale, atunci VM3, VM4 etc se configureaza identic cu masina secundara VM2.

Intai sa precizez faptul ca pentru instalarea urmatoarelor masini ai **2 variante**:

1. Din prima masina creezi o imagine martor (VHD) pe care ulterior o folosesti ca si sursa pentru instalarea celorlalte masini
2. Creezi fiecare masina "de la zero"

Am experimentat ambele situatii si am ajuns la concluzia ca **prima solutie** este cu adevarat utila doar daca nr. masinilor care urmeaza sa le instalezi este relativ mare. Am cateva argumente:

- folosirea primei variante elimina timpul necesar de configurare (SO, IIS, WebDeploy, Firewall etc) dar introduce alti timpi de asteptare, comparabili ca durata, cu operatii precum:
	- sysprep
	- creare imagine vhd martor
	- reprovizionare masina sursa (da, nu doar VM2 tb. create din vhd ci si VM1)
- daca vrei sa folosesti solutia cu VHD in ideia ca vei reduce timpul de reinstalare in cazul unui eventual defect, iti recomand sa te gandesti de 2 ori:
	- pastrarea si folosirea vhd-ului costa (30 GB stoacare + trafic)
	- timpul de reinstalare nu e critic atata timp cat vorbim de un WebFarm care are minim doua masini
- versiunile de Win.Srv.2012 din Azure Gallery se schimba destul de des (la 2 luni de la lansare a aparut deja varianta "Octombrie"). S-ar putea sa te trezesti ca ai pastrat VHD-ul dar o sa vrei sa folosesti o varianta actualizata.

Asadar, si a 2-a VM am sa o instalez tot "de la zero".

Portal: [https://manage.windowsazure.com/](https://manage.windowsazure.com/)

## Pas 1. Instalezi VM2 : ##

- Portal – VIRTUAL MACHINES – NEW
	- Selectezi, din Galerie, Windows Server 20012 (Oct.2012)
	- VM configuration:
		- VM Name: **AppsVm2**
		- Size: dupa necesitati si buget
	- VM mode:
		- **Connect to an existing** VM (selectezi din lista server-ul creat anterior – **AppsVm1**)
		- Storage Account: **appsvhds1** (creat in partea a II-a)
		- Reginon/AG: **AppsAffinity1** (aferent lui appsvhds1)
	- Availability Set:
		- Selectezi valoarea creata in partea a III-a: **AppsAvailable1**
- OBS: In  timpul provizionarii (aprox. 15 min), masina va trece succesiv prin starile:
	- Starting (Provisioning) – in acest timp va fi afectata temporar si starea primei masini
	- Stopped
	- Running (Provisioning)
	- Running (OK)

## Pas 2. Configurezi EndPoint-urile ##

- Dupa instalare, singurul port disponibil va fi cel de RDP (3389). Adauga inca o redirectare, ca in figura:
	- port 80 (HTTP) – Ataseaza-l la portul 80 al lui VM1 (Load-Balance). In felul acesta, portul 80 al ambelor masini va figura cu starea LB=YES
	- [port 8080 (WebDeploy)] – nu mai este nevoie de acest port fiindca publicarea codului pe VM2 se va face prin intermediul lui VM1 (vezi partea a V-a)
	- OBS: fiindca portul public 3389 este folosit deja de VM1, a trebuit sa-l schimb cu alta valoare (ex: 3390 sau altceva)

	![](https://dl.dropboxusercontent.com/u/43065769/blog/images/2012/EndPointsVM2.png)

## Pas3. Personalizezi SO (optional) ##

- CtrlPanel –> User Account -> Change User Account Settings (remove)
- ServerManager -> LocalServer –> IE Enhanced Security Configuration (off)
- Folder settings – unhide all

## Pas 4. Instalezi IIS ##

- Add Role: Web Server (IIS)
	- "Application Development" ->"ASP.NET 4.5"  si dependentele associate (fara asta nu poti rula o aplicatie ASP.NET Web API)
- Add Features:
	- .NET Framework 4.5 -> ASP.NET 4.5
	- Telnet Client (optional)
- Testeaza IIS-ul, dar si partea de balansare, din exterior (optional):
	- adauga in folder-ul corespunzator lui "Default Site" un fisier index.html  care sa afiseze numele server-ului:

		```
		<html><body>Server 2</body></html>
		```
 - In acest moment, o cerere catre [http://appsfarm1.cloudapp.net](http://appstudio.cloudapp.net/) ar trebui sa returneze, in mod aleator, numele celor doua servere din Farm. Atentie: probabilitatea de a obtine la interogarea de mai sus valori diferite creste daca folosim browsere diferite sau daca facem o pauza de cel putin 2 min. intre doua incercari successive.
	
		![](https://dl.dropboxusercontent.com/u/43065769/blog/images/2012/LoadBalanceDemo.png)

## Pas 5. Configurezi agentul WebDeploy ##

- Descarci (**nu** instalezi!) WebDeploy 3.0 de aici (link-ul x64) si salvezi intr-un folder
- Din directorul cu fisierul descarcat, lanseaza agentul de WebDeploy pe portul 8080, cu comanda:

	```
	msiexec /I webdeploy_amd64_en-us.msi /passive ADDLOCAL=ALL LISTENURL=http://+:8080/
	```

  ![](https://dl.dropboxusercontent.com/u/43065769/blog/images/2012/InstallWebDeploy.png)

- **Obs**: daca instalarea s-a facut cu success, in Program Files apare folder-ul "IIS"
- Deshide portul 8080 in Firewall:

 ![](https://dl.dropboxusercontent.com/u/43065769/blog/images/2012/FirewallOpenWebDeploy.png)

- **Obs2**: pe aceasta masina, WebDeploy va fi folosit doar pt. sincronizarea continutului folder-ului wwwroot cu server-ul principal (VM1).

In [partea a V-a](http://lucian.maran.ro/2012/11/26/configurarea-unui-web-farm-in-windows-azure-partea-v-publicarea-cu-webdeploy-din-visualstudio-2012/) urmeaza sa prezint componenta de deploy din VS2012 si,in [ultima parte](http://lucian.maran.ro/2012/11/26/configurarea-unui-web-farm-in-windows-azure-partea-vi-sincronizarea-automata-cu-webdeploy-si-powershell/), sincronizarea continutului publicat intre toate masinile din Farm.