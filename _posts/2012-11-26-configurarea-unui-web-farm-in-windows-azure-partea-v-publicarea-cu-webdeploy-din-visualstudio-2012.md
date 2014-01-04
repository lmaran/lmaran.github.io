---
layout: post
title:  "Configurarea unui Web Farm in Windows Azure (partea V - Publicarea cu WebDeploy din VisualStudio 2012)"
date:   2012-11-25 00:00:05
comments: true
categories: Dev-infrastructure Azure
---

- partea I – [Introducere](http://lucian.maran.ro/2012/11/25/configurarea-unui-web-farm-in-windows-azure-partea-i-introducere/)
- partea II – [Pregatirea infrastructurii](http://lucian.maran.ro/2012/11/25/configurarea-unui-web-farm-in-windows-azure-partea-ii-pregatirea-infrastructurii/)
- partea III – [Instalarea primei masini virtuale (VM1)](http://lucian.maran.ro/2012/11/25/configurarea-unui-web-farm-in-windows-azure-partea-iii-instalarea-primei-vm/)
- partea IV – [Instalarea unei masini virtuale secundare (VM2)](http://lucian.maran.ro/2012/11/25/configurarea-unui-web-farm-in-windows-azure-partea-iv-instalarea-unei-masini-virtuale-secundare/)
- partea V – Publicarea cu WebDeploy din VisualStudio 2012
- partea VI – [Sincronizare automata cu WebDeploy si PowerShell](http://lucian.maran.ro/2012/11/26/configurarea-unui-web-farm-in-windows-azure-partea-vi-sincronizarea-automata-cu-webdeploy-si-powershell/)

----------

## Pas 1. Creaza site-ul din IIS care urmeaza sa gazduiasca aplicatia: ##

- in *wwwroot*, creaza un folder nou (Ex: *api.idgenerator*...nu tb. sa fie neaparat identic cu numele site-ului)
- creaza un site in IIS, dupa modelul din figura:
 ![](https://dl.dropboxusercontent.com/u/43065769/blog/images/2012/SiteIISBlank.png)
- repeta acest pas, in mod identic, si pe celelalte servere secundare (Ex: VM2)

## Pas 2. Publica proiectul din Visual Studio 2012 ##

- asigura-te ca ai parcurs operatiile descrise in partile III si IV cu privire la:
	- **instalarea WebDeploy** pe cele 2 servere
	- **configurarea Firewall**-ului de pe cele 2 servere pt. accesul la portul 8080 (in aceasta etapa conteaza doar portul de pe VM1)
- site/application = numele site-ului creat anterior in IIS
- valideaza conexiunea (optional)
 ![](https://dl.dropboxusercontent.com/u/43065769/blog/images/2012/PublishWebDeploy.png)
- Publica si verifica rezultatul:
 ![](https://dl.dropboxusercontent.com/u/43065769/blog/images/2012/PublishOK.png)

## Pas 3. Testeaza rezultatul partial: ##

- Daca nu ai facut-o pana acum, **adauga o inregistrare** [de tip A](http://lucian.maran.ro/2012/11/23/dns-evita-sa-folosesti-cname/), **in DNS**, care sa mapeze numele site-ului la IP-ul **public** al WebFarm-ului. Acest IP public este identic pentru cele 2 masini  si se gaseste in portalul de management din Azure, la proprietatile oricarei din cele 2 masini.
- presupunand ca inregistrarea de mai sus a avut suficient timp sa se propage, apeleaza aplicatia (ex: [http://api.idgenerator.net](http://api.idgenerator.net/) )
- in acest moment ar trebui sa apara rezultatul dorit, returnat deocamdata doar de server-ul principal (VM1).

In [ultima parte](http://lucian.maran.ro/2012/11/26/configurarea-unui-web-farm-in-windows-azure-partea-vi-sincronizarea-automata-cu-webdeploy-si-powershell/) a acestui tutorial voi prezenta modalitatea in care codul ajuns pe primul server (VM1) se va propaga pe al 2-lea server (VM2) pentru a returna, in final, un raspuns balansat.