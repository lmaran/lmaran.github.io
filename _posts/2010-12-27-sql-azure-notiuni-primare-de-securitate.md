---
layout: post
title:  "SQL Azure - Notiuni primare de securitate"
date:   2010-12-27 00:00:01
comments: true
categories: SQLAzure
---

    GRANT / REVOKE / DENY <permissions> ON <securables> TO <principals>

Despre manipularea prin comenzi T-SQL a unor entitati din categoria *Permissions*, *Securables* sau *Principals*.


## 1. Obiecte (securables) ##
SQL este organizat sub forma unei colectii de obiecte.  Obiectele pot fi la nivel de instanta(server) sau la nivel de database.

Exemple de obiecte la nivel de database:

    SELECT * from sys.objects //conectat in prealabil la db "de lucru"

![](https://dl.dropboxusercontent.com/u/43065769/blog/images/2010/ObjectsSrv.png)

Exemple de obiecte la nivel de server:

    `SELECT * from sys.objects //conectat in prealabil la "master"`

![](https://dl.dropboxusercontent.com/u/43065769/blog/images/2010/Objectsdb.png)

Se observa ca este aceeasi interogare dar rulata in contexte diferite. Obiectele se mai numesc si 'securables' fiindca pe ele se aplica permisiunile.

### Schema ###

Fiecare obiect are un owner (se presupune ca nici un obiect nu a aparut din senin ci a fost creat de cineva). Daca acest owner este o persoana(utilizatorul) - asa cu era pana la SQL2005 -atunci aveai o pb. atunci cand trebuia sa stergi utilizatorul. Intr-o astfel de situatie, inainte de stergere trebuia sa reasignezi fiecare obiect la un alt user.

**Schema** s-a introdus ca un element intermediar intre user si obiect si nu este altceva decat un container de obiecte. Mai multe obiecte sunt grupate intr-o schema iar schema este detinuta de un user(owner). Bineinteles ca owner-ul schemei va fi si owner-ul obiectelor incluse in interiorul schemei. Astfel, daca vrei sa stergi un user care detine o schema, trebuie mai intai sa transferi dreptul de proprietate pt. respectiva schema catre un alt user.

    CREATE SCHEMA [Tenant1]  //Creaza Schema pt. noul tenant si o atribuie implicit utilizatorului "dbo" (principal_id=1)
    CREATE SCHEMA [Tenant1] AUTHORIZATION [Tenant1_User1]  //Creaza Schema pt. noul tenant si permite accesul la utilizatorului specificat
    ALTER USER [Tenant1_User1] WITH DEFAULT_SCHEMA = [Tenant1] //User-ul are acces default la noua schema
    SELECT * FROM sys.schemas //scheme si useri

![](https://dl.dropboxusercontent.com/u/43065769/blog/images/2010/Schema.png)

Daca initial ai creat schema fara optiunea AUTHORIZATION, implicit owner-ul schemei este dbo. Poti schimba acest lucru prin comanda:

    ALTER AUTHORIZATION ON SCHEMA::Tenant1 TO [Tenant1_User1]

Ca rezultat, coloana principal_id din poza de mai sus se modifica din 1 (dbo) in 6 (Tenant1_User1)

 
## 2. Permisiuni ##

    GRANT / REVOKE / DENY <permissions> ON <securables> TO <principals>

    GRANT EXECUTE TO db_executor //atribuie permisiunea de 'EXECUTE' la rolul 'db_executor'
    GRANT SELECT ON SCHEMA::Tenant1 TO Tenant1_User1 //atribuie permisiunea de SELECT pe obiectele din schema
    DENY VIEW DEFINITION TO Tenant1_User1 //interzice lui User1 sa vada metadata
    REVOKE VIEW DEFINITION FROM Tenant1_User1 //reface posibilitatea ca User1 sa vada metadata
    REVOKE SELECT ON SCHEMA::Tenant1 FROM Tenant1_User1 //revoca permisiunea aferenta

Inainte de SQL2005, permisiunile se alocau numai pe obiecte. Incepand cu SQL2005, permisiunile p. fi alocate la nivel de server, database, schema sau obiect(tabela, view, SP etc). Daca atribui o permisiune la un anumit nivel (ex. database) atunci respectiva permisiune va fi valabila si pt. nivelele inferioare (ex: schema si toate obiectele din schema)

**Impersonarea** – iti permite sa operezi in numele altui utilizator. Pt. aceasta tb. sa ai permisiunea IMPERSONATE. 

    EXECUTE AS USER 'Tenant1_User1' //ruleaza urm. comenzi ca si cand ai fi autentificat cu 'Tenant1_User1'
    EXECUTE AS LOGIN 'Tenant1_Login1'
    REVERT //revine la user-ul/login-ul initial

## 3. Principals ##

- pt. server (Logins + Server Roles (fixe))
- pt. database (Database Users + Loginless users + Database Roles (fixe sau create de utilizator))

**Conturi de Login** - pt. acces la server. Se ruleaza conectat in prealabil la db Master

    CREATE LOGIN Tenant1_Login1 WITH PASSWORD = 'StrongPsw1'
    ALTER LOGIN [Tenant1_Login1] WITH NAME = [Tenant1_Login11] //redenumeste login name
    SELECT * FROM sys.sql_logins //vizualizare conturi de login

![](https://dl.dropboxusercontent.com/u/43065769/blog/images/2010/sql_logins.png)

Un cont de login poate avea unul din urm. 2 **Server Roles**:

- loginmanager – gestioneaza conturile de utilizator si de login de pe server
- dbmanager – permisiuni maxime (echiv. cu contul initial de SQL Azure)


**Utilizatori** – pt. acces la database: Urm. comenzi se ruleaza conectat in prealabil la db "de lucru"

    CREATE USER Tenant1_User1 FROM LOGIN Tenant1_Login1 //Creare login pt. acces la db.

Daca se doreste ca user-ului sa i se ataseze o alta schema decat cea default (adica dbo) si aceasta schema este deja creata, atunci:

    CREATE USER Tenant1_User1 FROM LOGIN Tenant1_Login1 WITH DEFAULT_SCHEMA = 'Tenant1'
    SELECT USER_NAME() //afiseaza numele utilizatorului curent (user context)
    SELECT * FROM sys.database_principals //vizualizare useri (tip S[SQL]) si roluri R[Roluri]) //SQL Azure suporta doar auth. cu useri SQL, nu si Windows Auth.

![](https://dl.dropboxusercontent.com/u/43065769/blog/images/2010/UseriSiRoluri.png)

**Roluri** (optional - in caz ca vrei sa atribui utilizatorului un rol inexistent):
Vom crea mai jos un rol numit db_executor caruia ii vom atribui permisiunea de EXECUTE. In final vom atribui acest rol lui Tenant1_User1 a.i. acesta sa poata rula doar proceduri stocate. Atat.

    CREATE ROLE db_executor //implicit, default schema se pune pe NULL iar owning_principal_id =1 (userul dbo)
    GRANT EXECUTE TO db_executor //de ex. daca vrei sa ruleze SP-uri
    SELECT * FROM sys.database_permissions //vizualizeaza permisiunile alocate pe useri sau roluri

![](https://dl.dropboxusercontent.com/u/43065769/blog/images/2010/Permissions.png)

    EXEC sp_addrolemember 'db_executor', 'Tenant1_User1'; //atribuie rolul la un utilizator
    EXEC sp_droprolemember 'db_executor', 'Tenant1_User1' //sterge rol
    SELECT * FROM sys.database_role_members //afiseaza rolurile utilizatorilor

![](https://dl.dropboxusercontent.com/u/43065769/blog/images/2010/RoleMembers.png)

Rol 7= db_executor; rol 16384=db_owner

## Concluzii ##
Conceptele si comenzile din SQL Azure sunt asemanatoare celor din varianta "On-premise". Putem spune ca sunt chiar aceleasi, dar intr-o forma  trunchiata. Un aspect diferit este contextul in care trebui rulate anumite comenzi, context care nu poate fi schimbat prin comanda "USE" ca  si in varianta clasica si presupune realizarea de conexiuni diferite la db. "Master" (pt. comenzile care manipuleaza obiecte de tip server) respectiv conexiuni la db. “de lucru” (pt. comenzi ce vizeaza obiecte specifice unei anumite baze de date).

Detalii [aici](http://mscerts.programming4.us/sql_server/SQL%20Azure%20%20%20Security%20-%20Access%20Control.aspx).

