---
layout: post
title:  "Custom Domain in GithubPages"
date:   2014-01-08 00:00:01
comments: true
categories: jekyll
---

Procedura oficiala este descrisa [aici](https://help.github.com/articles/setting-up-a-custom-domain-with-pages).
Concret, in cazul blog-ului personal (un site de tipul *[User&Organization Page](https://help.github.com/articles/user-organization-and-project-pages)*), pasii parcursi au fost:

## Configurare site Jekyll (Github repository) ##

- adauga un fisier cu numele `CNAME` in radacina repository-ului
  ![](https://dl.dropboxusercontent.com/u/43065769/blog/images/2014/cname.png)
- adauga in fisierul CNAME domeniul dorit (`maran.ro`). 
  ![](https://dl.dropboxusercontent.com/u/43065769/blog/images/2014/cname-content.png)

## Configurare DNS ##

- folosesc DNSimple pe post de DNS provider. Inregistrarile sunt:
  ![](https://dl.dropboxusercontent.com/u/43065769/blog/images/2014/dnsimple-config.png)
- relevante sunt doar cele 2 inregistrari: tip `A` si `CNAME`. 
- inreg. de tip `URL` este una mai speciala si este de fapt o redirectare (301) facuta direct de catre serverul de DNS. Github Pages [nu suporta](http://stackoverflow.com/a/10766694) decat:
	- **o singura** inreg. de tip **A**
	- **o singura** inreg. de tip **CNAME** 
- asadar, daca vrei sa adaugi si alte subdomenii in afara de "*www*" (ex: *lucian.maran.ro*) atunci esti nevoit sa apelezi la un HTTP redirect, ca in poza de mai sus.
