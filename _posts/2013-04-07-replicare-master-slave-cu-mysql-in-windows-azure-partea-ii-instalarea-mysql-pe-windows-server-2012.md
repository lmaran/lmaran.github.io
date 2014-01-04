---
layout: post
title:  "Replicare Master-Slave cu MySql in Windows Azure (partea II - Instalarea MySql pe Windows Server 2012)"
date:   2013-04-07 00:00:02
comments: true
categories: Dev-infrastructure Azure MqSql
---

Procedura de instalare este descrisa f. bine [aici](http://www.windowsazure.com/en-us/manage/windows/common-tasks/install-mysql/) pentru Windows Server 2008 R2 si MySQL 5.5. Pentru Windows Server 2012 si MySQL 5.6 diferentele sunt nesemnificative.

Singurul loc unde am intampinat probleme a fost la Windows Firewall. A trebuit sa deschid manual portul Public pe 3306 fiindca installer-ul de MySQL a deschis doar variantele Private si Domain ale respectivului port.

1: Instalezi VM folosind imaginea de *Windows Server 2012*

![](https://dl.dropboxusercontent.com/u/43065769/blog/images/2013/InstalareMysql1.png)

2: Stabilesti numele si configuratia:

![](https://dl.dropboxusercontent.com/u/43065769/blog/images/2013/InstalareMysql2.png)

3: Introduci numele sub care masina MySQL va fi accesata din exterior. Selectezi apoi un storage account, adica locul unde va fi depozitat blob-ul ce va contine drive-ul C.

![](https://dl.dropboxusercontent.com/u/43065769/blog/images/2013/InstalareMysql3.png)

4: Odata masina creata adaugi un disk virtual de 5GB (un blob, viitorul drive F) in care urmeaza sa stochezi datele din MySQL

![](https://dl.dropboxusercontent.com/u/43065769/blog/images/2013/InstalareMysql4.png)

5: Stabilesti dimensiunea viitorului disk (ex: 5GB)

![](https://dl.dropboxusercontent.com/u/43065769/blog/images/2013/InstalareMysql5.png)

6: Te conectezi remote la VM si atasezi disk-ul recent creat la SO (Disk Management)

![](https://dl.dropboxusercontent.com/u/43065769/blog/images/2013/InstalareMysql6.png)

7: La final verifici ca disk-ul este formatat:

![](https://dl.dropboxusercontent.com/u/43065769/blog/images/2013/InstalareMysql7.png)

8: De pe server pornesti IE si descarci installer-ul (NU kit-ul) de MySQL Community Edition (daca nu ai cont poti sa-ti faci unul gratuit)

![](https://dl.dropboxusercontent.com/u/43065769/blog/images/2013/InstalareMysql8.png)

9: Pornesti instalarea:

![](https://dl.dropboxusercontent.com/u/43065769/blog/images/2013/InstalareMysql9.png)

10: Daca este o masina de productie instaleaza “Server Only” si nu uita sa redirectezi instalarea catre disk-ul atasat mai sus (F):

![](https://dl.dropboxusercontent.com/u/43065769/blog/images/2013/InstalareMysql10.png)

11: Verifica bifa care iti deschide portul 3306 pe firewall:

![](https://dl.dropboxusercontent.com/u/43065769/blog/images/2013/InstalareMysql11.png)

12: Daca esti pe Master, poti sa adaugi deja user-ul sub care va rula procesul de replicare:

![](https://dl.dropboxusercontent.com/u/43065769/blog/images/2013/InstalareMysql12.png)

13: Si gata:

![](https://dl.dropboxusercontent.com/u/43065769/blog/images/2013/InstalareMysql13.png)

14: Installer-ul de MySQL deschide pe firewall doar profilele Public si Domain pt. portul 3306:

![](https://dl.dropboxusercontent.com/u/43065769/blog/images/2013/InstalareMysql14.png)


15: Daca vrei sa accesezi MySQL-ul din afara Azure (ex: din propriul LAN, pt. administrare) atunci deschide si profilul Public:

![](https://dl.dropboxusercontent.com/u/43065769/blog/images/2013/InstalareMysql15.png)

16: Adauga din portalul de Azure un Endpoint care sa permita accesul din exterior catre VM (initial este permis doar accesul pt. conectare remote – RDP):

![](https://dl.dropboxusercontent.com/u/43065769/blog/images/2013/InstalareMysql16.png)

17: Tot traficul receptionat de Azure LoadBalancer pe 3306 va fi redirectat la portul 3306 al VM:

![](https://dl.dropboxusercontent.com/u/43065769/blog/images/2013/InstalareMysql17.png)

18: In acest moment s-a finalizat instalarea server-ului MySQL. Poti testa conexiunea spre acest serviciu ruland un TELNET de pe o statie din exterior.

OBS: daca testezi din LAN-ul local (Intranet) atunci asigura-te ca accesul spre 3306 nu iti este blocat de firewall-ul companiei.

![](https://dl.dropboxusercontent.com/u/43065769/blog/images/2013/InstalareMysql18.png)

19: Iar raspunsul, daca totul este ok, va fi de forma:

![](https://dl.dropboxusercontent.com/u/43065769/blog/images/2013/InstalareMysql19.png)
