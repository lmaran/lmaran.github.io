---
layout: post
title: "Configurarea unui Web Farm in Windows Azure (partea VI - Sincronizarea automata cu WebDeploy si PowerShell)"
date: 2012-11-26 00:00:06
comments: true
categories: Dev-infrastructure Azure
---

- partea I – [Introducere](http://lucian.maran.ro/2012/11/25/configurarea-unui-web-farm-in-windows-azure-partea-i-introducere/)
- partea II – [Pregatirea infrastructurii](http://lucian.maran.ro/2012/11/25/configurarea-unui-web-farm-in-windows-azure-partea-ii-pregatirea-infrastructurii/)
- partea III – [Instalarea primei masini virtuale (VM1)](http://lucian.maran.ro/2012/11/25/configurarea-unui-web-farm-in-windows-azure-partea-iii-instalarea-primei-vm/)
- partea IV – [Instalarea unei masini virtuale secundare (VM2)](http://lucian.maran.ro/2012/11/25/configurarea-unui-web-farm-in-windows-azure-partea-iv-instalarea-unei-masini-virtuale-secundare/)
- partea V – [Publicarea cu WebDeploy din VisualStudio 2012](http://lucian.maran.ro/2012/11/26/configurarea-unui-web-farm-in-windows-azure-partea-v-publicarea-cu-webdeploy-din-visualstudio-2012/)
- partea VI – Sincronizare automata cu WebDeploy si PowerShell

---

Pana in aceasta ultima etapa am configurat cele 2 servere si am facut publicarea codului pe server-ul principal. Mai ramane de vazut cum facem ca acest cod sa fie distribuit si pe restul masinilor din WebFarm. As avea mai multe optiuni:

1. din Visual Studio, sa [public](http://lucian.maran.ro/2012/11/26/configurarea-unui-web-farm-in-windows-azure-partea-v-publicarea-cu-webdeploy-din-visualstudio-2012/) **repetat** (manual) codul pe fiecare server (tot cu _WebDeploy_ dar cu un _EndPoint_ adecvat pentru fiecare masina)
2. din Visual Studio, sa [public](http://lucian.maran.ro/2012/11/26/configurarea-unui-web-farm-in-windows-azure-partea-v-publicarea-cu-webdeploy-din-visualstudio-2012/) **o singura data** codul pe server-ul principal iar apoi continutul folder-ului _wwwroot_ de pe fiecare server sa-l mentin sincronizat prin diferite metode:

- cu [Dropbox](https://www.dropbox.com/) sau alte servicii online ce ofera sincronizare prin intermediul unui repository extern
- cu [SyncToy](http://www.microsoft.com/en-us/download/details.aspx?id=15155) sau alte aplicatii dedicate pentru sincronizare la nivelul LAN-ului local
- am avut si ideia cu un vhd memorat intr-un blob si atasat pe post de disk comun, dar am testat si am vazut ca solutia asta nu e posibila (cel putin nu cu VM-urile dn Azure)
- sa folosesc folosesc functia de sincronizare care sta la baza serviciului [WebDeploy](http://www.iis.net/downloads/microsoft/web-deploy).

Am decis sa folosesc pentru sincronizare WebDeploy pentru ca acesta este un agent pe care il am deja instalat pe server-ul principal (il folosesc pentru publicarea din VS2012).

Cu alte cuvinte, atat transferul datelor din Visual Studio catre server-ul master (VM1) cat si propagarea datelor de la acesta catre serverele secundare (VM2, etc) au la baza acelasi agent si acelasi protocol. Cred ca nu mai trebuie sa spun ca, in aceste cazuri, copierea datelor se face doar prin transmiterea diferentelor.

**Concret:**

Voi descrie in continure modul in care se face sincronizare datelor intre folder-ul wwwroot al unui server central (VM1) si acelasi folder, de pe doua sau mai multe servere secundare (VM2, etc)

## Pas 1. Asigura-te ca ai parcurs operatiile descrise in partile III si IV cu privire la:

- **instalarea WebDeploy** pe cele 2 servere
- **configurarea Firewall**-ului de pe cele 2 servere pt. accesul la portul 8080 (in aceasta etapa conteaza doar portul de pe VM2)

## Pas 2. Creaza scriptul de sincronizare: (doar pe masina sursa)

- creaza pe server-ul master (VM1) un folder (ex: C:\SyncScript)
- creaza un fisier text (ex: sync.ps1) in care sa adaugi urmatorul script de tip PowerShell:

  ```
  $source = (gc env:computername).toLower() + ":8080" # masina sursa este masina pe care ruleaza acest script

  $target = "appsvm3:8080" # numele si portul masinii destinatie

  $exe = "C:\Program Files\IIS\Microsoft Web Deploy V3\msdeploy.exe"
  [Array]$params = "-verb:sync", "-source:contentPath=C:\Inetpub\wwwroot,computerName=$source", "-dest:contentPath=C:\Inetpub\wwwroot,computerName=$target";
  & $exe $params;
  ```

- scriptul de mai sus sincronizeaza doar doua masini si l-am adaptat pentru nevoile personale (nu foloseste si nu necesita instalarea unei biblioteci externe)
- daca vrei sa sincronizezi mai mult de 2 masini, trebuie sa rulezi scriptul de mai sus intr-o bucla.
- daca vrei o varianta ce ofera maxim de flexibilitate (determina automat numele tuturor serverelor ce compun WebFarm-ul) atunci descarca biblioteca [Windows Azure PowerShell Cmdlets](http://go.microsoft.com/?linkid=9811175&clcid=0x409) si apeleaza direct scriptul dupa care m-am inspirit si eu.

## Pas 3. Adauga permisiuni pentru rularea scriptului: (doar pe masina sursa)

- [Aici](http://technet.microsoft.com/en-us/library/ee176949.aspx) se precizeaza urmatoarele: "_By default, PowerShell’s execution policy is set to **Restricted**; that means that scripts – including those you write yourself – won’t run. Period._"
- lansezi consola de PowerShell si introduce comanda care acorda permisiunile necesare:
  - **Set-ExecutionPolicy RemoteSigned**
- optional, lansezi comanda care iti arata nivelul actual de securitate:
  - **Get-ExecutionPolicy**
- in urma executiei primei comenzi, nivelul de securitate ar trebui sa scada de la "_Restricted_" la "_RemoteSigned_"

## Pas 4. Testeaza manual functionalitatea scriptului: (doar pe masina sursa)

- lansezi, din "Command Prompt (Admin)" comanda:

```
powershell.exe -File C:\SyncScript\sync.ps1
```

- ca rezultat, cele doua folderere implicate in sincronizare vor avea acelasi continut

## Pas 5. Planifica rularea periodica a scriptului de mai sus: (doar pe masina sursa)

- Creaza un TaskSchedule, astfel:

  - Start Program: powershell.exe

  ```
  Add arg: -File C:\SyncScript\sync.ps1
  ```

- Bifeaza: "_run whether user is logged on or not_"
- Bifeaza: "_Repeat every 1 minute_" (sau dupa necesitati)
  ![](/assets/images/2012/ScheduleGeneral.png)

![](/assets/images/2012/ScheduleTriggers.png)

![](/assets/images/2012/ScheduleActions.png)

- din acest moment, orice modificare aparuta in folder-ul wwwroot de pe server-ul principal (VM1) va fi captata si retransmisa automat pe server-ul secundar (VM2)

Asta a fost tot!

## Live Demo

[http://api.idgenerator.net/](http://api.idgenerator.net/) (probabilitatea de a obtine valori diferite creste daca folosim browsere diferite, daca facem o pauza de cel putin 2 min. intre doua incercari successive sau daca tinem F5 apasat timp de 2-3 secunde)

**Referinte:**

Maarten Balliauw – [Setting up a webfarm using Windows Azure Virtual Machines](http://blog.maartenballiauw.be/post/2012/06/10/Setting-up-a-webfarm-using-Windows-Azure-Virtual-Machines.aspx)
Michael Washam – [Publishing and Synchronizing Web Farms using Windows Azure Virtual Machines](http://michaelwasham.com/2012/08/13/publishing-and-synchronizing-web-farms-using-windows-azure-virtual-machines/)
