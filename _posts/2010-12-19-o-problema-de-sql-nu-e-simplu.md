---
layout: post
title: "O problema de SQL...nu e simplu!"
date: 2010-12-19 00:00:02
comments: true
---

Am avut nevoie, pt. niste rapoarte in CRM, sa creez un dataset care sa expuna date din tabele (sau view-uri) legate intre ele prin metadata. Simplificand si renuntand la contextul “CRM”, problema se poate defini astfel:

**Se dau** urmatoarele tabele:

![](/assets/images/2010/ProblamaSQL.png)

**Se cere** un query\* SQL care sa returneze\*\* asta:

![](/assets/images/2010/SeCereSQL.png)

\*Am spus “query” dar se poate folosi orice structura de programare care ramane in granita SQL (Proceduri stocate, functii, cursori, dynamic SQL sau chiar cod CLR scris cu C#/Vb)

\*\* Am simplificat problema dar, evident, algoritmul tb. sa functioneze - nemodificat – si pt. cazul general cand am mai multe tabele de tip "Clienti" (Clienti1, Clienti2...ClientiN). Cu alte cuvinte, nu pot presupune cunoscute numele si numarul acestor tabele.

Rezolvarea am gasit-o punand cap la cap informatii din mai multe surse, astfel:

![](/assets/images/2010/SolutieSQL.png)

Cunoaste cineva o solutie mai simpla? Sau mai performanta? (cursorii stiu ca sufera la acest capitol).
