---
layout: post
title:  "DNS: Evita sa folosesti CNAME!"
date:   2012-11-23 00:00:01
comments: true
categories: Dev-infrastructure Azure
---

De ce? Pentru ca iti pasa de performanta aplicatiei tale. Simplu, rezolutia de nume prin "**CNAME**" este sensibil mai lenta decat rezolutia directa prin "**A**" Record. Perfect explicabil, daca ne gandim ca prima metoda implica cel putin o interogare de DNS suplimentara fata de a doua.

Am [monitorizat](https://www.pingdom.com/), pe o perioada de 20 de ore, cu o cadenta de 1 minut, acelasi site, folosind cele 2 tipuri de inregistrari:

**Caz 1:**  *www.idgenerator.net*  ->  *appsfarm1.cloudapp.net* (**CNAME** record)

![](https://dl.dropboxusercontent.com/u/43065769/blog/images/2012/cname-rec.png)

**Caz 2:**  *api.idgenerator.net*  ->  *168.63.64.179* (**A** record)

![](https://dl.dropboxusercontent.com/u/43065769/blog/images/2012/a-rec.png)

In al 2-lea caz, timpul de raspuns s-a dovedit **cu 40% mai mic**. Desigur, cu cat creste complexitatea paginii, ponderea timpului alocat rezolutiei de nume, scade. Ramane insa o valoare absoluta de **50-100 ms** care nu merita ignorata.

PS: Da, chiar si pt. VM-urile gazduite in **Azure** poti renunta la CNAME. Si asta pentru ca IP-ul public (VIP-ul) se aloca doar in faza de deploy, nu si la restart/upgrade.

Raman insa scenarii cand inregistrarile de tip CNAME sau ALIAS raman singura solutie, dar nu vreau sa ma abat de la subiect.