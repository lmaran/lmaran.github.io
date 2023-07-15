---
layout: post
title: "VS2012 - Remote Debugging"
date: 2012-09-19 00:00:01
comments: true
categories: VisualStudio
---

De ce **Remote** Debugging?

Scenariul ar fi urmatorul... am doua aplicatii:

- app1 = REST API (backend), **subdomainA**.eta2u.ro;
- app2 = GUI (frontent), **subdomainB**.eta2u.ro

App2 consuma datele furnizate de App1. In mediile de **test** sau de **productie** cele 2 aplicatii ruleaza in domenii diferite, comunicatia dintre ele bazandu-se pe specificatiile [CORS](http://en.wikipedia.org/wiki/Cross-origin_resource_sharing). Cand insa cele 2 aplicatii sunt lansate pe masina de **dezvoltare**, domeniul sub care ruleaza este unul comun (_localhost_), caz in care codul specific CORS nu se mai executa sau se comporta diferit.

Asa am ajuns sa depanez codul unei aplicatii (app1) in timp ce aceasta se executa pe o alta masina decat cea locala.

## 1. Instalare componenta remote (Windows Server 2012)

- download Remote Tools for Visual Studio 2012 (x86 sau X64)
- ruleaza fisierul descarcat pe server (masina remote). Aplicatia apare in "_Programs and Features_" iar fisierele aferente in "_C:\Program Files\Microsoft Visual Studio 11.0_"
- Optional, se creaza un shortcut sau "Pin to Taskbar" pt. fisierul **msvsmon** din folder-ul "_C:\Program Files\Microsoft Visual Studio 11.0\Common7\IDE\Remote Debugger\x64_"
- lanseaza in executie fisierul de mai sus, cu setarile implicite si, neaparat "**run as administrator**". In final aplicatie e pregatita sa accepte conexiuni de la Visual Studio-ul local:

![](/assets/images/2012/waitingforconn.png)

## 2. Configurare componenta locala (Windows 8):

- Deschide solutia/proiectul aferent aplicatiei remote (aceeasi versiune a codului)
- Configureaza "**symbols**":
  - Pentru debug este nevoie pe server de fisierul din proiect cu extensia .pdb. - La un WebDeploy de tip "**Release**" sunt copiate pe server doar fisierele strict necesare rularii aplicatiei - La un WebDeploy de tip "**Debug**" pe langa fisierele strict necesare rularii aplicatiei, pe server este copiat si fisierul .pdb (in folder-ul "_bin_")
    ![](/assets/images/2012/webDeploy.png)
  - Adauga in VS2012 adresa fizica, de pe server, a fisierului de symbol-uri (Tools – Options – Debugging):
    ![](/assets/images/2012/symbolAdr.png)
- defineste un "**breakpoint**"
- lanseaza procesul de debugging: Tools – **Attach to Process**:

![](/assets/images/2012/attach.png)

## Observatii:

- valoare pt. "Qualifier" am introdus-o cu litere mari, chiar in acest ecran (cu aceeasi denumire, in sectiunea de "Find" nu a gasit nimic...poate fiindca server-ul nu se afla in acelasi subnet?!)
- **w3wp.exe** apare doar daca bifezi "Show process from all users"
- **w3wp.exe** nu apare daca aplicatia de pe server este "idle" (nu a mai fost folosit in ultima perioada; AppPool=down)
- daca aplicatia de pe server (msvsmon) nu ai lansat-o ca si administrator, vei primi un mesaj de eroare sugestiv. Daca totul e ok, pe server vezi conexiunile initiate de pe VS2012 local.:

![](/assets/images/2012/connected.png)

In final, daca apelezi aplicatia care ruleaza pe server-ul remote, atunci ar trebui sa se activeze breakpointul dorit:

![](/assets/images/2012/demofinal.png)

**UPDATE:** Nu uita ca dupa ce ai terminat operatiunea de debug pe server, sa reinstalezi versiunea de tip "Release" sau sa modifici manual debug="false". Dave Ward [explica](http://encosia.com/a-harsh-reminder-about-the-importance-of-debug-false/) de ce.
