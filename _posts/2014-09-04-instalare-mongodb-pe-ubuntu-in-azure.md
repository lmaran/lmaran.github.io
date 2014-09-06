---
layout: post
title:  "Instalare MongoDB pe Ubuntu, in Microsoft Azure"
date:   2014-09-04 00:00:01
comments: true
categories: MongoDB, Ubuntu, Azure
---

## Introducere ##

Desi cu cunostinte limitate in lumea "Linux", am decis totusi sa gazduiesc MongoDB pe un SO ce functioneaza cu resurse hardware minimale - Ubuntu. Ideia era sa incep cu instanta minima (A0 - 768 MB), urmand sa o upgradez pe masura ce traficul si dimensiunea bazelor de date cresc.

Un alt argument ar fi pretul direct. La aceleasi resurse (RAM + CPU), o masina Linux este substantial (~25%) mai ieftina decat un server Windows.

Versiunile instalate:

- Ubuntu 14.04.1 LTS (release 25.07.2014)
- MongoDB 2.6.4 (release 11.08.2014)

## 1. Creaza o noua masina virtuala (VM) folosind portalul Azure ##

Pasii ii presupun cunoscuti (detalii [aici](http://azure.microsoft.com/en-us/documentation/articles/virtual-machines-linux-tutorial/)) dar am sa adaug cateva observatii:

- am folosit un Virtual Network fiindca vreau sa pot accesa masina si din "interior" (de catre alta VM) - folosind o adresa privata.
- am deschis endpoint-urile SSH (22) si MongoDB (27017). Recomandari de securitate:
	- deschide portul de Mongo doar daca intentionezi sa accesezi Mongo din exterior (Robomongo?)
	- modifica valorile porturilor de pe interfetele publice (random)
	- seteaza ACL pt. fiecare port a.i. sa controlezi adresele IP de pe care poti accesa aceste servicii

## 2. Instaleaza Putty si acceseaza remote masina ##

- instaleaza Putty [manual](http://www.chiark.greenend.org.uk/~sgtatham/putty/download.html) sau folosind [Chocolatey](http://chocolatey.org/packages/putty)
- in fereastra de conectare poti alege:
	- HostName = numele ales pt. "CloudService" (nu numele server-ului). Ex: mongosrv.cloudapp.net
	- Port = endpoint-ul public (valoarea 'random' aleasa in etapa precedenta)
	- Conn.Type = SSH
- login pe masina cu usr/psw setate in timpul provizionarii VM

![](https://dl.dropboxusercontent.com/u/43065769/blog/images/2014/09-04-ubuntu1.png)

## 3. Instaleaza MongoDB ##

Daca folosesti procedura de instalare recomandata de MongoDB, fisierele de date si log (si binarele, bineinteles) se vor stoca implicit pe disk-ul C. Nu poti modifica acest lucru in timpul instalarii  dar o poti face ulterior.

Pasii sunt explicati pe larg [aici](http://docs.mongodb.org/manual/tutorial/install-mongodb-on-ubuntu/). Eu am sa notez doar secventa de comenzi:

```
sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv 7F0CEB10
echo 'deb http://downloads-distro.mongodb.org/repo/ubuntu-upstart dist 10gen' sudo tee /etc/apt/sources.list.d/mongodb.list
sudo apt-get update
sudo apt-get install mongodb-org
```

## 4. Ruleaza MongoDB ##

Reporneste **serviciul** care ruleaza pe server (`mongod`):

```
sudo service mongod stop
sudo service mongod start
```

OBS - fara etapa de `start` si `stop` primeam eroare la conectare: "Connection refused".

Lanseaza **client**-ul de interogare (`mongo`):

```
mongo
```

Ca si test, afiseaza lista cu DB existete:`show dbs` sau help-ul cu toate celelalte comenzi: `help`

![](https://dl.dropboxusercontent.com/u/43065769/blog/images/2014/09-04-ubuntu2.png)

## 5. Ataseaza un nou disk la VM ##

Se recomanda ca datele si log-urile din Mongo sa stea pe un alt disk decat cel dedicat pt. SO. Asadar, din portalul de Azure se ataseaza masinii un nou disk sub forma unui blob.
Detalii [aici](http://azure.microsoft.com/en-us/documentation/articles/virtual-machines-linux-tutorial/#attachdisk). 

## 6. Partitioneaza, formateaza si monteaza noul disk in Ubuntu ##

Am sa prezint in continuare o "compilatie" personala inspirata din mai multe  surse: [doc1](http://azure.microsoft.com/en-us/documentation/articles/virtual-machines-linux-tutorial/), [doc2](https://help.ubuntu.com/community/InstallingANewHardDrive), [doc3](http://docs.mongodb.org/ecosystem/tutorial/install-mongodb-on-linux-in-azure/).

- Verifica lista cu disk-urile instalate (ca sa stii ce disk sa partitionezi).

    ```
sudo lshw -C disk
    ```

    In Azure, primul disk atasat va fi, de regula, al 3-lea disk (ex: \dev\sdc):

- Partitioneaza disk-ul identificat anterior (voi pp. in continuare ca noul disk = \dev\sdc)

 ```
sudo fdisk /dev/sdc  
 ```  
 
 - type "n" -> creaza o noua partitie
 - type "p" --> primary partition
 - enter --> partition 1
 - enter --> first sector (default)
 - enter --> last sector (default) - cream o sg. partitie pt. tot disk-ul
 - type "w" --> salveaza modificarile pe disk (si exit)


- Optional. Partitia mai sus creata va fi `/dev/sdc1`. (Are un "1" in plus fata de disk). Poti verifica numele acestei partitii cu:

 ```
 sudo fdisk /dev/sdc
 ```

	- type "p" (de la print)


- Formateaza partitia /dev/sdc1 cu formatul "ext4":

 ```
sudo mkfs -t ext4 /dev/sdc1
 ```

- Verifica partitia nou creata si copiaza-i UUID-ul (vei avea nevoie de el ulterior):

 ```
sudo -i blkid
 ```

- Creaza folder-ul la care urmeaza sa montezi noul drive. [Aici](https://help.ubuntu.com/community/InstallingANewHardDrive) se recomanda ca acest folder sa se numeasca `/media`:

 ```
sudo mkdir /media/mongodrive
 ```

- Monteaza partitia `/dev/sdc1` la folder-ul anterior creat:

 ```
sudo mount /dev/sdc1 /media/mongodrive
 ```

 - `sudo umount /media/mongodrive` - daca trebuie sa faci unmount


- Optional. Verifica drive-ul nou montat:

 ```
sudo df -h
 ```

- Configureaza ca remontarea disk-ului sa se faca automat dupa reboot. Acest lucru se face prin editarea fisierului `/etc/fstab`. **Atentie!** - daca editezi ceva incorect sistemul nu mai booteaza. Deschide fisierul cu editorul "nano":

 ```
sudo nano -w /etc/fstab
 ```

- Adauga la sfarsitul fisierului linia:

 ```
UUID=5e1ccfd5-e58a-456f-a78d-84bb5cb09fea   /media/mongodrive   ext4   defaults   0   2
 ```

 - unde valoare lui UUID se obtine cu comanda `sudo -i blkid` (prezentata anterior)
	 - ctrl+o (save)
	 - enter (overwrite)
	 - ctrl+x (exit)


- Optional. Restarteaza sistemul (`sudo reboot`) si asigura-te ca disk-ul s-a montat automat (`sudo df -h`)

 ![](https://dl.dropboxusercontent.com/u/43065769/blog/images/2014/09-04-ubuntu3.png)

## 7. Redirecteaza MongoDB catre noul drive ##

Doar datele si log-ul. Detalii [aici](http://askubuntu.com/a/257724).

- Editeaza fisierul `/etc/mongodb.conf`:

 ```
sudo nano -w /etc/mongod.conf
 ```

 - Editeaza urmatorii parametri:
	- `dbpath=/media/mongodrive/data`
	- `logpath=/media/mongodrive/mongod.log`
	- `bind_ip=0.0.0.0` (sau comenteaza linia) - implicit Mongo nu permite accesul decat de pe masina locala)


- Creaza folder-ul 'data'

 ```
sudo mkdir /media/mongodrive/data
 ```

- Schimba owner-ul la folder-ul 'mongodrive' (root --> mongodb)

 ```
sudo chown -R mongodb:mongodb /media/mongodrive/
 ```
  - `root` = contul sub care rulezi atunci cand esti loginat in consola 
  - `mongodb` = contul sub care ruleaza implicit serviciul mongodb


- Optional. Poti vedea noile drepturi cu:

 ```
cd /media/mongodrive
ls -l
 ```

 ![](https://dl.dropboxusercontent.com/u/43065769/blog/images/2014/09-04-ubuntu4.png)

- Restarteaza serviciul `mongod`. Asta trebuie facut dupa orice modificare a fisierului `mongod.conf`:

 ```
sudo service mongod restart
 ```

- Optional. Rulezi inca odata comanda `ls -l` (din folder-ul `/media/mongodrive`) si ar trebui sa apara deja fisierul `mongod.log`. Totodata, in folder-ul `data` ar trebui sa apara primele fisiere de mongo:

 ```
cd data
ls -l
 ```

 ![](https://dl.dropboxusercontent.com/u/43065769/blog/images/2014/09-04-ubuntu5.png)

 - daca acum creezi o DB noua, fisierul aferent va aparea tot in aceast folder, de pe noul disk.

## 8. Concluzie ##

Instalarea si configurarea unei instante de MongoDB nu este la fel de simpla pe Linux ca si pe Windows dar, pentru situatiile in care platesti hosting-ul "din buzunar", merita efortul.