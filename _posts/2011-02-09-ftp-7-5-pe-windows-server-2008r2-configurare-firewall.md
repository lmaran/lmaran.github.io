---
layout: post
title:  "FTP 7.5 pe Windows Server 2008R2 - configurare firewall"
date:   2011-02-09 00:00:02
comments: true
categories: IT-tools
---

## 1. Instalare FTP 7.5 ##
Win. srv.2008 (IIS 7.0) vine cu varianta FTP 6.0 (legacy). Poti instala manual FTP 7.5 (disponibil ca un pachet independent ce p. fi descarcat de [aici](http://www.iis.net/downloads/microsoft/ftp)).

Win. srv. 2008 R2 (IIS 7.5) vine cu varianta FTP 7.5.

Etapa de instalare este [simpla](http://www.iis.net/learn/install/installing-publishing-technologies/installing-and-configuring-ftp-7-on-iis-7) - sar peste ea.


## 2. Configurare FTP 7.5 (setari initiale) ##
[Detalii](http://www.iis.net/learn/publish/using-the-ftp-service/creating-a-new-ftp-site-in-iis-7) pt. instalare New FTP Site, sau

[Detalii](http://www.iis.net/learn/publish/using-the-ftp-service/adding-ftp-publishing-to-a-web-site-in-iis-7) pt. instalare Adding FTP to a Web Site (optional)

## 3. Configurare FTP  7.5 (setari avansate) ##
Pentru a putea accepta si cereri de conectare din partea unor clienti care activeaza in spatele propriului lor router sau firewall, voi configura serverul sa accepte conexiuni FTP Pasive. Diferenta dintre FTP Activ si FTP Pasiv este este explicata [aici](http://www.velikan.net/iis-passive-ftp/).

Va trebui deci ca pe server sa deschidem porturile:

- port 21
- 1..n porturi consecutive(>1023), unde n=nr. max. de conexiuni pe care doresti sa le accepti simultan

![](https://dl.dropboxusercontent.com/u/43065769/blog/images/2011/firewallFtp.png)

Numarul si adresa celor n porturi consecutive le stabilesc din IIS -> ComputerSettings (not FTPSite Setting) ->FTP Firewall Support

![](https://dl.dropboxusercontent.com/u/43065769/blog/images/2011/OpenPorts.png)

## 4. Configurare client: ##

![](https://dl.dropboxusercontent.com/u/43065769/blog/images/2011/tcmd.png)

Pt. debug se p. folosi log-ul de pe serverul FTP de la adresa C:\inetpub\logs\LogFiles\FTPSVC3.