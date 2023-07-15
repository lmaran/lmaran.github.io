---
layout: post
title: "Windows 8 - instalare .NET Framework 3.5 (3.0, 2.0)"
date: 2012-09-03 00:00:01
comments: true
categories: Dev-tools, Windows-8
---

Windows 8 vine cu .NET Framewor 4.5. Sunt insa aplicatii care necesita versiuni mai vechi ale acestui framework. SQL Management Studio 2012 si Borg (aplicatia de gestiune a celor de la Transart) sunt primele de care m-am lovit:

![](/assets/images/2012/borgErr.png)

In mod normal, instalarea acestei componente se face cu "_Turn Windows Feature on_":

![](/assets/images/2012/err.png)

In background, pentru a descarca fisierele necesare, procesul de instalare apeleaza la serviciul de "Windows Update". Daca acest “canal” este operational, atunci fisierele sunt descarcate iar instalarea se finalizeaza fara probleme.

Dar nu intotdeauna putem apela la aceasta metoda fiindca putem avea si situatia in care accesul la Windows Update este obstructionat (spre exemplu cand politicile de domeniu forteaza ca update-urile sa se faca printr-un serviciu intermediar – gen WSUS):

![](/assets/images/2012/errMsg.png)

Rezolvarea (cel putin in cazul meu) am gasit-o parcurgand pagina cu indicatii la care se face referire in imaginea de mai sus. Concret, am rulat comanda:

```
Dism.exe /online /enable-feature /featurename:NetFX3 /All /Source:D:\sources\sxs /LimitAccess
```

unde "D" este drive-ul in care am pus kit-ul de Windows 8:

![](/assets/images/2012/CmdOk.png)

In final am obtinut confirmarea ca versiunea dorita de .NET Framework s-a instalat correct:

![](/assets/images/2012/NetFxOk.png)
