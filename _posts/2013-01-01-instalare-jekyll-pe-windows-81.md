---
layout: post
title:  "Instalare Jekyll pe Windows 8.1"
date:   2014-01-01 00:00:01
comments: true
categories: jekyll
---


## Conditii preliminare

Asigura-te ca ai deja instalat **Ruby**, **RubyGems** si **RubyDevKit**. Pentru detalii poti consulta precedentul articol.

## Instalare

Jekyll se poate instala ca si un Gem independent, dar prefer ca instalarea lui sa se faca prin pachetul **github-pages**. In felul acesta, pe langa Jekyll se instaleaza si toate dependentele necesare, rezultand astfel un mediu de productie cat mai apropiat de Github.
    
`gem install github-pages`

 ![](https://dl.dropboxusercontent.com/u/43065769/blog/images/2014-01-01-installed-github-pages-cmd.png)

Observatii:
- in timpul instalrii vor aparea cateva mesaje de eroare de tipul "_unable to convert "\x90" from ASCII-8BIT to UTF-8 for..._" dar se pot ignora fiindca se refera doar la pachetele de documentatie
- se vor instala astfel 25 de gem-uri. Aceste se pot vedea in folder-ul _"C:\Ruby200-x64\lib\ruby\gems\2.0.0\gems"_ (cele din figura, minus cele 4 marcate ca fiind deja instalate default)

 ![](https://dl.dropboxusercontent.com/u/43065769/blog/images/2014-01-01-installed-github-pages.png)

## Verificarea instalarii

- `jekyll new myblog` creaza un folder numit myblog care va contine structura de fisiere a unui blog minimal
- `cd myblog`
- `jekyll serve` converteste structura de foldere/fisiere intr-un site static. Acesta se va depune in folder-ul _"_site"_.

 ![](https://dl.dropboxusercontent.com/u/43065769/blog/images/2014-01-01-testing-jekyll.png)

  - OBS: daca primesti un mesaj de eroare legat de libraria **pygments** poti sa mergi in fisierul "_config.yml" (din radacina blog-ului) si sa setezi `pygments = false`. Pygments este o librarie Python folosita pt. formatarea codului.

- In final, site-ul functional se poate verifica la adresa: "_http://localhost:4000_":
 ![](https://dl.dropboxusercontent.com/u/43065769/blog/images/2014-01-01-testing-jekyll-url.png)