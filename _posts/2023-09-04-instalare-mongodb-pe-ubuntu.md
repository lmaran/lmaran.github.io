---
layout: post
title:  "Instalare MongoDB pe Ubuntu"
date:   2023-09-04 19:44:29 +0100
categories: nodejs
---

# Instalare pe Ubuntu 20.04.5 (Focal Fossa): - detalii aici (oficial) ai aici (Digital Ocean)

OBS1: Aceeasi procedura ca la Nginx
OBS: Mongo este inclus in Ubuntu software repository dar cu o versiune mai veche. Ca sa instalam ultima versiune de tb. sa adaugam si Mongo repo la lista de repo-uri folosita de "apt-get"

copy the following repositories to the end of ` /etc/apt/sources.list`:
echo "deb [ arch=amd64,arm64 ] http://repo.mongodb.org/apt/ubuntu xenial/mongodb-org/3.4 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-3.4.list
	
Ca sa nu primesti un warning in timp ce descarci pachetele => adauga un key cu care urmeaza sa semnezi pachetele:

sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv 0C49F3730359A14518585931BC711F9BA15703C6

sudo apt-get update
sudo apt-get install -y mongodb-org

# Testeaza versiunea instalata: 
    
OBS: Prin procedura de mai sus se instaleaza 2 componente (ambele cu aceeasi versiune):
mongod - procesul care ruleaza server-ul de mongo (MongoDB server)
mongo - clientul care se conecteaza la server (MongoDB shell)

Imediat dupa instalare se poate testa doar clientul (server-ul tb. pornit manual)
mongo  --> 3.4.6 

- OBS:  imediat dupa instalare da un mesaj de eroare la conectare la 127.0.0.1:27017
- Cauza: implicit mongo nu permite accesul remote. 
- Solutie:   --> comenteaza linia "bind_ip = 127.0.0.1"  Inlocuieste 127.0.0.1 cu 0.0.0.0
    - daca modifi ac. fisier => sudo service mongod restart


# Run MongoDB server (procesul mongod)

sudo service mongod start (imi apare ca ar fi deja rulat, poate ca nu mai e nevoie)

    - testeaza ca serviciul de mongo este pornit, ca mai jos

# Testeaza ca serviciul-ul este pornit:
	
- met.1: te conectezi din Studio 3T (remote)
- met 2: in fisierul "/var/log/mongodb/mongod.log" tb. sa apara linia:
    [initandlisten] waiting for connections on port <port>


# Config MongoDB server to run as a service: (detalii aici)

- implicit server-ul de mongo nu ruleaza ca si serviciu.
- ca sa ruleze ca si serviciu tb. sa configurezi Systemd (varianta superioara pt. Upstart)
- vezi procedura de instalare de pe pag. oficiala.


## a. ruleaza din nou serviciul (cu systemctl)

sudo systemctl start mongodb
    
## b. testeaza daca serviciul a pornit ok (tb. sa apara cu status = active (running)):

sudo systemctl status mongodb
    
## c. seteaza ca servicil sa porneasca automat (la restart):

sudo systemctl enable mongodb

# Testeaza ca serviciul de mongo este pornit (ca mai sus).

## Manage Mongod service: (dupa ce l-ai instalat ca si serviciu ca mai sus)

sudo systemctl stop mongodb 
sudo systemctl start mongodb 
sudo systemctl start mongodb
	
## Setari default: - detalii aici (oficial) ai aici (Digital Ocean)

/var/lib/mongodb --> data files
/var/log/mongodb --> log files
/etc/mongod.conf --> config file (daca modifici acest fisier => sudo service mongod restart)

# Uninstall
	Vezi procedura de pe site-ul mongodb

# Update Mongo

Pe server:
- Am incercat intai sa reinstalez noua versiune peste cea veche cf: http://docs.mongodb.org/manual/tutorial/install-mongodb-on-ubuntu/ dar in final mi-a raportat tot vechea versiune.
- In final, am facut un backup (mongodump --out backup-path), am sters vechea versiune (inclusiv fisierele, cf. aceluias link) si am reinstalat

# Debug Mongo Install

- Pb.1: dupa ce am rulat prima oara "mongo" am primit msg:
Failed global initialization: BadValue Invalid or no user locale set. Please ensure LANG and/or LC_* environment variables are set correctly.
Sol.: fa ce scrie aici: http://stackoverflow.com/a/30383910, adica am adaugat cele 2 linii la sf. fisierului + "sudo reboot"

- Pb.2: dupe ce am depasit Pb.1, la rularea "mongo" am primit 2 sugestii referitoare "transparent_hugepage"
Sol: fa ce scrie aici: https://docs.mongodb.org/master/tutorial/transparent-huge-pages/i 
+  "sudo service mongod restart"


Ca sa verifica ca mongo ruleaza OK => type "mongo". Daca primesti warning-urile de mai jos (ref. la transparent_hugepage) atunci poti sa le ignori sau le poti rezolva asa cum se spune aici:
http://stackoverflow.com/a/29181918/2726725 
la mine a functionat.
