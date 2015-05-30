---
layout: post
title:  "Instalare Jekyll pe Windows 8.1"
date:   2014-01-01 00:00:01
comments: true
categories: jekyll
---


**Update (30.05.2015)** 

Am reinstalat Jekyll folosind [bundler](http://bundler.io) (in loc de `gem install github-pages`), asa cum e descris [aici](https://help.github.com/articles/using-jekyll-with-pages/#installing-jekyll).
In continuare pot apela:

- `bundle install` - pt a instala automat toate gem-urile declarate in fisierul `Gemfile`
- `bundle update` si `bundle uninstall` - idem pt. update/uninstall
- `bundle exec jekyll serve` - ruleaza site-ul

Ca rezultat, pachetele instalate (ruleaza `github-pages versions`) au fost identice ca versiune cu [pachetele folosite pe github](https://pages.github.com/versions/).

----

## Instalare

Asigura-te ca ai deja instalat **Ruby**, **RubyGems** si **RubyDevKit**. Pentru detalii poti consulta [precedentul articol](http://maran.ro/2013/12/31/instalare-ruby-rubygems-si-rubydevkit-pe-windows-81-64bit/).

Jekyll se poate instala ca si un Gem independent, dar prefer ca instalarea lui sa se faca prin pachetul **github-pages**. In felul acesta, pe langa Jekyll se instaleaza si toate dependentele necesare, rezultand astfel un mediu de productie cat mai apropiat de Github.
    
`gem install github-pages`

 ![](https://dl.dropboxusercontent.com/u/43065769/blog/images/2014/installed-github-pages-cmd.png)

Observatii:
- in timpul instalrii vor aparea cateva mesaje de eroare de tipul "_unable to convert "\x90" from ASCII-8BIT to UTF-8 for..._" dar se pot ignora fiindca se refera doar la pachetele de documentatie
- se vor instala astfel 25 de gem-uri. Aceste se pot vedea in folder-ul _"C:\Ruby200-x64\lib\ruby\gems\2.0.0\gems"_ (cele din figura, minus cele 4 marcate ca fiind deja instalate default)

![](https://dl.dropboxusercontent.com/u/43065769/blog/images/2014/installed-github-pages.png)

## Verificarea instalarii

- `jekyll new myblog` creaza un folder numit myblog care va contine structura de fisiere a unui blog minimal
- `cd myblog`
- `jekyll serve` converteste structura de foldere/fisiere intr-un site static. Acesta se va depune in folder-ul _"_site"_.

![](https://dl.dropboxusercontent.com/u/43065769/blog/images/2014/testing-jekyll.png)

  OBS: daca primesti un mesaj de eroare legat de libraria **pygments** poti sa mergi in fisierul "_config.yml" (din radacina blog-ului) si sa setezi un alt sintax highlighter. Ex: `highlighter: rouge`. Pygments este o librarie Python folosita pt. formatarea codului. Daca vrei sa o folosesti, asigura-te ca ai Python instalat (Python 2.7 fiindca cu Python 3 se pare ca sunt [ceva probleme](http://stackoverflow.com/questions/17364028/jekyll-on-windows-pygments-not-working)) si ca exista o cale in PATH catre python.exe.

- In final, site-ul functional se poate verifica la adresa: "_http://localhost:4000_":

![](https://dl.dropboxusercontent.com/u/43065769/blog/images/2014/testing-jekyll-url.png)

## Monitorizarea automata a modificarilor (optional)

**UPDATE** (23.11.2014) - in versiunile mai noi, monitorizarea automata se face implicit (nu mai este necesar param. `-w`)

- `jekyll serve -w` lansat cu parametrul _-w (watch)_, serviciul Jekyll reconstruieste site-ul "on-the-fly", pe masura ce apar modificari.

 OBS: necesita in prealabil instalarea unui pachet ruby: `gem install wdm`. Windows Directory Monitor (WDM) este o librarie care detecteaza modificarile aparute in directoare.

![](https://dl.dropboxusercontent.com/u/43065769/blog/images/2014/jekyll-watch.png)

In acest moment este suficient sa dai un refresh la browser daca vrei sa pre-vizualizezi modificarile facute (nu mai trebuie sa repornesti Jekyll-ul).

## Comenzi utile

- `github-pages versions` - afiseaza versiunea tuturor pachetelor folosite de `github-pages versions`. In final, poti compara rezultatul cu valorile din [aceasta lisa](https://pages.github.com/versions/).

![](https://dl.dropboxusercontent.com/u/43065769/blog/images/2014/2014-11-23-github-pages-versions.jpg)