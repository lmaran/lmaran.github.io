---
layout: post
title:  "Configurarea unui Web Farm in Windows Azure (partea III - Instalarea primei VM)"
date:   2012-11-25 00:00:03
comments: true
categories: Dev-infrastructure Azure
---

- partea I – [Introducere](http://lucian.maran.ro/2012/11/25/configurarea-unui-web-farm-in-windows-azure-partea-i-introducere/)
- partea II – [Pregatirea infrastructurii](http://lucian.maran.ro/2012/11/25/configurarea-unui-web-farm-in-windows-azure-partea-ii-pregatirea-infrastructurii/)
- partea III – Instalarea primei masini virtuale (VM1)
- partea IV – [Instalarea unei masini virtuale secundare (VM2)](http://lucian.maran.ro/2012/11/25/configurarea-unui-web-farm-in-windows-azure-partea-iv-instalarea-unei-masini-virtuale-secundare/)
- partea V – [Publicarea cu WebDeploy din VisualStudio 2012](http://lucian.maran.ro/2012/11/26/configurarea-unui-web-farm-in-windows-azure-partea-v-publicarea-cu-webdeploy-din-visualstudio-2012/)
- partea VI – [Sincronizare automata cu WebDeploy si PowerShell](http://lucian.maran.ro/2012/11/26/configurarea-unui-web-farm-in-windows-azure-partea-vi-sincronizarea-automata-cu-webdeploy-si-powershell/)

----------

Prima masina virtuala dintr-un Farm (VM1) are un rol mai special, in sensul ca ea stabileste EndPoint-urile principale sau sursa de date (Master) pentru sincronizarea cu celelalte masini.

Portal: [https://manage.windowsazure.com/](https://manage.windowsazure.com/)

## Pas 1. Instalezi VM1: ##

Portal – VIRTUAL MACHINES – NEW

- Selectezi, din Galerie, Windows Server 20012 (Oct.2012)
- VM configuration:
	- VM Name: **AppsVm1**
	- Size: dupa necesitati si buget
- VM mode:
	- **StandAlone** VM
	- DNS Name: **AppsFarm1**.cloudapp.net (numele sub care va fi vazut, la final, Farm-ul)
	- Storage Account: **appsvhds1** (creat in partea a II-a)
	- Reginon/AG: **AppsAffinity1** (aferent lui appsvhds1)
- Availability Set:
	- Create new: **AppsAvailable1**

- OBS: In timpul provizionarii (aprox. 15 min), masina va trece succesiv prin starile:
	- Starting (Provisioning)
	- Stopped
	- Running (Provisioning)
	- Running (OK)

## Pas 2. Configurezi EndPoint-urile ##

Dupa instalare, singurul port disponibil va fi cel de RDP (3389). Adauga inca doua redirectari, ca in figura:

- port 80 (HTTP) – Normal EndPoint (**fara** Load-Balance) pt. VM1 (starea LB=Yes apare automat dupa ce atasezi si VM2 la acelasi port)
- port 8080 (WebDeploy) -portul pe care urmeaza sa fac WebDeploy din VS2012

![](https://dl.dropboxusercontent.com/u/43065769/blog/images/2012/AzureVM1EndPoint.png)

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
- Testeaza IIS-ul din exterior (optional)
	- Adauga in folder-ul corespunzator lui "Default Site" un fisier index.html  care sa afiseze numele server-ului:

  		`<html><body>Server 1</body></html>`

	- In acest moment, o cerere catre [http://appsfarm1.cloudapp.net](http://appstudio.cloudapp.net/) ar trebui sa returneze numele server-ului. Chiar daca in acest moment IIS-ul de pe VM2 nu este configurat, Load Balancer-ul sesizeaza ca la cererile catre portul 80 de pe VM2 nu primeste raspuns si astfel redirecteaza toate cererile catre VM1.

## Pas 5. Configurezi agentul WebDeploy ##

- Descarci (**nu** instalezi!) WebDeploy 3.0 de [aici](http://www.iis.net/downloads/microsoft/web-deploy) (link-ul x64) si salvezi intr-un folder
- Din directorul cu fisierul descarcat, lanseaza agentul de WebDeploy pe portul 8080, cu comanda:

   ` msiexec /I webdeploy_amd64_en-us.msi /passive ADDLOCAL=ALL LISTENURL=http://+:8080/`

	![](https://dl.dropboxusercontent.com/u/43065769/blog/images/2012/InstallWebDeploy.png)
- **Obs**: daca instalarea s-a facut cu success, in Program Files apare folder-ul "IIS"
- Deshide portul 8080 in Firewall:
	![](https://dl.dropboxusercontent.com/u/43065769/blog/images/2012/FirewallOpenWebDeploy.png)

sau:

    netsh advfirewall firewall add rule name="My WebDeploy" dir=in action=allow protocol=TCP localport=8080

- **Obs2**: vom folosi WebDeploy atat pt. a receptiona publicarile facute din VS2012 cat si ulterior, pt. sincronizarea continutului folder-ului wwwroot intre VM1 si VM2.

In[ partea a IV-a](http://lucian.maran.ro/2012/11/25/configurarea-unui-web-farm-in-windows-azure-partea-iv-instalarea-unei-masini-virtuale-secundare/) urmeaza sa prezint diferentele care apar la configurarea unei masini virtuale secundare.