---
layout: post
title: "Replicare Master-Slave cu MySql in Windows Azure (partea I - Introducere)"
date: 2013-04-07 00:00:01
comments: true
categories: Dev-infrastructure Azure MqSql
---

## Se da:

- o masina MySQL care ruleaza OnPremise si deserveste o aplicatie importanta, in continua crestere.

## Se cere (pentru respectiva masina):

- toleranta la defecte
- si performanta
- ...fara a sari cu costurile in aer

## Solutia:

![](/assets/images/2013/MySqlAzure1.png)

Am sa justific arhitectura aleasa plecand de la cerinte:

- **Hardware** tolerant la defecte –> **Windows Azure**
  - MySQL este instalat pe o masina virtuala in Azure (VM)
  - toate disk-urile atasate unei VM sunt stocate in blob-uri externe cu [tripla redundata](http://blogs.msdn.com/b/windowsazurestorage/archive/2011/11/20/windows-azure-storage-a-highly-available-cloud-storage-service-with-strong-consistency.aspx) (automatic failover, zero downtime)
  - suplimentar, cele 2 blob-uri de pe Master le-am configurat cu [Geo-Replicare](http://blogs.msdn.com/b/windowsazurestorage/archive/2011/09/15/introducing-geo-replication-for-windows-azure-storage.aspx) (protectie la calamitati)
  - VM sunt si ele "regenerate" automat. In caz de defect, AppFabric provizioneaza o noua VM, cu aceeasi configuratie, urmand ca in final sa-i ataseze acesteia blob-urile apartinand initial masinii defecte (_~15 min. downtime_).
- **Software**
  - Windows Server 2012 Datacenter Ed. (pret inclus in costul masinii)
  - MySql Community Server 5.6 (ultima varianta, gratuit)
  - [MySql Workbench](http://dev.mysql.com/downloads/workbench/5.2.html) (optional) – instalat pe PC local, pt. administrarea ambelor servere (gratuit)
- **Performanta** –> **Master/Slave Replication**
  - Slave-ul degreveaza Master-ul de operatia de Backup periodic
  - Slave-ul deserveste modulele de Rapoarte sau alte cereri Rd/Only ale aplicatiei.
- **Costuri** aproximative (configuratie medie)
  - Cu Windows Azure, resursele pot creste pe masura cerintelor -> platesc doar cat am nevoie
  - **115** $/luna pt. Medium VM (2 x 1.6GHz CPU, 3.5GB RAM) (0.16$/h \* 720h/luna).
  - **115** $/luna – daca se doreste si al 2-lea server, pt. replicare
  - **9.50** $/luna – Storage pt. 100GB/luna (Geo Redundant).
  - **12** $/luna – banda de transfer de 100GB/luna. Alte preturi [aici](http://www.windowsazure.com/en-us/pricing/overview/?fb=en-us)
    - inbound traffic to Azure = free
    - inbound/outbound in interiorul aceluias Datacenter = free (Ex: replicare Master -> Slave)

In [partea a 2-a](http://maran.ro/2013/04/07/replicare-master-slave-cu-mysql-in-windows-azure-partea-ii-instalarea-mysql-pe-windows-server-2012/) poti citi despre instalarea unui MySQL pe o VM din Azure.
