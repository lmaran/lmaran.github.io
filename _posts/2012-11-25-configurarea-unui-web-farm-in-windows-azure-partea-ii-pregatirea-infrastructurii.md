---
layout: post
title: "Configurarea unui Web Farm in Windows Azure (partea II - pregatirea infrastructurii)"
date: 2012-11-25 00:00:02
comments: true
categories: Dev-infrastructure Azure
---

- partea I – [Introducere](http://lucian.maran.ro/2012/11/25/configurarea-unui-web-farm-in-windows-azure-partea-i-introducere/)
- partea II – Pregatirea infrastructurii
- partea III – [Instalarea primei masini virtuale (VM1)](http://lucian.maran.ro/2012/11/25/configurarea-unui-web-farm-in-windows-azure-partea-iii-instalarea-primei-vm/)
- partea IV – [Instalarea unei masini virtuale secundare (VM2)](http://lucian.maran.ro/2012/11/25/configurarea-unui-web-farm-in-windows-azure-partea-iv-instalarea-unei-masini-virtuale-secundare/)
- partea V – [Publicarea cu WebDeploy din VisualStudio 2012](http://lucian.maran.ro/2012/11/26/configurarea-unui-web-farm-in-windows-azure-partea-v-publicarea-cu-webdeploy-din-visualstudio-2012/)
- partea VI – [Sincronizare automata cu WebDeploy si PowerShell](http://lucian.maran.ro/2012/11/26/configurarea-unui-web-farm-in-windows-azure-partea-vi-sincronizarea-automata-cu-webdeploy-si-powershell/)

---

Portal: [https://manage.windowsazure.com](https://manage.windowsazure.com)

## Pas 1. Creezi un Affinity Group (AG):

Portal – NETWORKS – AFFINITY GROUPS – NEW

Name: AppsAffinity1

OBS:

- Nu folosi un AG creat cu vechiul portal (Silverlight) fiindca, asa cum se arata si [aici](http://social.msdn.microsoft.com/Forums/windowsazure/en-US/b8706433-9728-43b1-b1a9-b0e98496901d/affinity-group-name-as-guid-new-portal?forum=windowsazuremanagement), pot aparea unele probleme:
  - Numele AG-ului apare urmat de un GUID;
  - Nu poti crea o VM cu un storage creat cu acest AG;
  - **update**: [o parte](http://social.msdn.microsoft.com/Forums/windowsazure/en-US/b8706433-9728-43b1-b1a9-b0e98496901d/affinity-group-name-as-guid-new-portal?forum=windowsazuremanagement) din probleme s-au rezolvat;
- Un AG este atasat de o anumita subscriptie. Deci nu vei putea face un Farm cu VM din subscriptii diferite.
- Mai departe, toate resursele (VM, StorageAccount-ul pt. VHD-uri, StorageAccount-urile cu datele utile) vor fi create cu AG-ul de mai sus.

## Pas 2. Creezi un Storage Account (SA):

Portal - STORAGE – NEW

- Name: **AppsVhds1**
- Region/AffinityGroup: **AppsAffinity1**
- Geo-Replication: **False**. Am decis sa nu platesc suplimentar pentru date care pot fi restaurate, adica:
  - imaginea initiala cu Windows Server 2012
  - setarile aferente (IIS, WebDeploy, etc)
  - codul aplicatiilor (folder-ul wwwroot)

## Utile:

- subscriptia MSDN Premium [ofera](http://www.windowsazure.com/en-us/pricing/member-offers/msdn-benefits/) storage de 40GB Local si 40 GB Geo-Redundant
- subscriptia Partner Network [ofera](http://www.windowsazure.com/en-us/offers/ms-azr-0002p) storage de 35GB Local si 35 GB Geo-Redundant
- cele 2 tipuri de redundanta nu se compenseaza la plata. Si pe factura apar ca pozitii diferite. Altfel spus, daca nu folosesti spatial Geo-Redundant, il pierzi...nu iti 'capitalizeaza' contul de storage Local.
- o imagine initiala de Windows Server 2012 ocupa 30GB
- chiar daca cele 2 discuri virtuale aferente lui VM1 si VM2 ocupa in blob 2\*30=60GB, se taxeaza doar spatiul de date **efectiv ocupat** pe respectivele partitii. In cazul meu, fara niciun site, drive-ul C de pe ambele servere ocupa 10.5GB. Asta inseamna 21GB/luna sau 0.7 GB/zi:

![](/assets/images/2012/AzureSorageSamplePrice.png)

## Concluzie:

Am creat asadar un **Affinity Group** sub care urmeaza sa fie grupate toate resursele ulterioare si apoi un **Storage Account** in care voi stoca imaginile VHD corespunzatoare celor doua VM care formeaza WebFarm-ul.

In [partea a III-a](http://lucian.maran.ro/2012/11/25/configurarea-unui-web-farm-in-windows-azure-partea-iii-instalarea-primei-vm/) instalez si configurez prima masina virtuala.
