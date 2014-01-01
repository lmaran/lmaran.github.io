---
layout: post
title:  "Instalare Ruby, RubyGems si RubyDevKit pe Windows 8.1-64bit"
date:   2013-12-31 00:00:01
comments: true
categories: jekyll
---


### Instalare Ruby si RubyGems

#### 1. Download si instalare

Metoda recomandata este prin [Ruby installer](http://rubyinstaller.org). Acesta instaleaza atat Ruby cat si Gem-ul (un package manager pt. Ruby). 

- am folosit versiunea [Ruby 2.0.0-p353 (x64)](http://dl.bintray.com/oneclick/rubyinstaller/rubyinstaller-2.0.0-p353-x64.exe?direct)
- instalarea se lanseaza automat la download
- instalare in calea default (*C:\Ruby200-x64*), toate cele 3 bife activate
![](https://dl.dropboxusercontent.com/u/43065769/blog/images/2013-12-31-installed-ruby.png)

#### 2. Verificarea instalarii

- `ruby -v` verifica instalarea ruby
(cda se poate lansa din cmd, din orice folder. Asta pt. ca una din cele 3 bife de mai sus seteaza variabila PATH. **Nu uita** sa restartezi consola cmd dupa instalarea Ruby, sau dupa orice modificare a variabilei PATH)
- `gem -v` verifica instalarea Gem-ului

#### 3. Verificarea ultimei versiuni

- `gem -v` verifica ver. instalata de Gem. Aceasta se p. compara cu ultima versiune aflata la [rubygems.org](http://rubygems.org)
- `gem update --system` updateaza Gem-ul, daca e cazul

### Instalare Ruby DevKit

#### 1. Download si instalare

- **Atentie!** - versiunea de DevKit tb. sa fie in concordanta cu versiunea de Ruby, asa cum se specifica [aici](http://rubyinstaller.org/downloads)
- pt. "_Ruby 2.0.0-p353 (x64)_" am folosit [DevKit-mingw64-64-4.7.2-20130224-1432-sfx.exe](http://cdn.rubyinstaller.org/archives/devkits/DevKit-mingw64-64-4.7.2-20130224-1432-sfx.exe)
- copierea kit-ului se initiaza direct din download. Eu am folosit pt. extragere calea: "_C:\Ruby200-x64-DevKit_" (aceeasi cale ca si Ruby + _"-DevKit"_)
- ruleaza din cmd: 

 1. `cd C:\Ruby200-x64-DevKit`
 2. `ruby dk.rb init` (ca rezultat se creaza fisierul "_C:\Ruby200-x64-DevKit\config.yml_" care va contine calea catre folder-ul de ruby, adica "_- C:/Ruby200-x64_")
 3. `ruby dk.rb install`
 
  ![](https://dl.dropboxusercontent.com/u/43065769/blog/images/2014-01-01-installed-rubydevkit.png)

#### 2. Testare instalarii

- `gem install json --platform=ruby`
- `ruby -rubygems -e "require 'json'; puts JSON.load('[42]').inspect"` 
 - daca rezultatul returnat in console este `[42]` => Ruby poate folosi corect DevKit-ul
 - **folder**-ul in care se instaleaza toate Gem-urile este _"C:\Ruby200-x64\lib\ruby\gems\2.0.0\gems"_. Aici se va regasi si pachetul recent instalat (_Json_)

----------

### Comenzi Gem uzuale (bonus)

Am facut o lista cu cele mai uzuale comenzi. Este strictul de necesar pe care l-am folosit incercand sa-mi instalez Jekyll-ul. O descriere completa gasesti [aici](http://guides.rubygems.org/command-reference/).

**Install/Uninstal**l cmds:
- `gem install mysql`
- `gem install mysql --pre` (pre-release)
- `gem uninstall mysql`

**List/Search** cmds:
- `gem list` - list all installed gems
- `gem search mysql` - search for a specific gem

**Update/Clean** cmds:
- `gem -v` - get gem version
- `gem update --system` - actualizeaza package manager-ul
- `ruby update` - updateaza toate gem-urile
- `gem update mysql` - upd un anumit gem
- `gem outdated` - gems that are now outdated (they have a newer version available)
- `gem clean` - remove outdated versions of gems that are installed, leaving the new updated version installed