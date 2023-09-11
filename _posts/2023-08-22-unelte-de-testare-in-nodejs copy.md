---
layout: post
title:  "Unelte de testare în NodeJs"
date:   2023-08-22 11:39:17 +0300
categories: nodejs
---

# Unit test frameworks:

## Mocha

- cel mai folosit framework pt. Node (6300 stars, f. activ)
- folosit de Rob Conery (Pluralsight)
- Nu include un assertion library. Tb. sa incluzi unul (vezi o lista mai jos)
- Pluralsight curs f. interesant: http://www.pluralsight.com/courses/unit-testing-nodejs (Node, chai, sinon, Istambul)

## Jasmine

- f cunoscut pt. Angular dar e folosit si cu node (8530 stars, activ)
- folosit de Shawn W (Pluralsight)
- Include un assertion library dar, daca nu-ti place, poti folosi una din lib. e mai jos.
- mai slab decat mocha d.p.d.v al testelor asincrone
- Nu include un test runner (ex: se poate folosi Karma sau Chutzpah)

## Vows

- 1430 stars, slab mentinut (3 months ago)
- Comparatii: Articol1 (03.2015): autorul alege Jasmine + Karma + PhantomJS

# Expectation/assertion libraries:

## Chai

- 1700 stars, f. activ
- folosit in Alexander Zanfir (Pluralsight)

## Should.js

- 1986 stas, activitate medie(16 days ago)
-folosit in Yeoman Angular fullStack, Rob Conery (Pluralsight), Alexander Zanfir (Pluralsight)


# End-to-end (functional) testing:

Daca ai doar API (fara front end):

- hippie
- supertest - testeaza un server http

Daca ai si front-end:

## Protractor (3600 stars, 6 days)

- Gandit pt. aplicatii Angular
- Testele se scriu in javascript (Node.js) si CSS
- Se bazeaza pe Selenium WebDriver API
- Ruleaza testele folosind un browser real (la fel ca un user obisnuit)
- Scris de Google + Angular team
    
## Nightwatch (2698, 2 months)

- Nu are suport pt. Angular
- Testele se scriu in javascript (Node.js) si CSS
- Se bazeaza pe Selenium WebDriver API

$ Headless web testing - browser-ul ruleaza fara GUI

Aceste teste sunt utile daca vrei sa rulezi teste e2e pe un server unde nu poti instala un full-browser (Ex: un Linux fara UI). Practic sunt tot niste browsere, dar fara UI, dar care au l abaza engine-ul browser-ului original ()WebKit/Gecko). Detalii aici

## PhantomJS (13500 stars, 13 days ago)

- testeaza UI-ul folosindu-se de WebKit (Chrome, Safari)
  
## SlimerJS  (1350 Stars, a month ago)

- SlimerJS allows you to interact with a web page through an external JS script:
  - Opening a webpage,
  - Clicking on links,
  - Modifying the content...
- testeaza UI-ul folosindu-se de Gecko (FF) 
- not yet truly headless

## CasperJS (4100 stars, 13 days ago)

- is a navigation scripting & testing utility for PhantomJS and SlimerJS written in Javascript
		

# Mocking:

- Sinon JS
- Nock - 1950 stars, f. activ

Articol interesant (04.2014) despre un proces complet de testing +tools. Autorul foloseste:

- Mocha - test framework
- ShouldJS - assertion library
- Sinon JS - mocking
- Nock - mock http request
- Supertest - test Http Server + Express app.
- Rewire - dependency injector
- Istablul - code coverage
- JSHint - coe analysis
- JSCS - code style checker
  - …si la final da sectiunea de configurare din package.json pt. toate tool-urile de mai sus

Articol2 (07.2014) => mocha + chai + sinon
Articol3 (02.2015) argumenteaza de ce a ales Mocha + Chai + Supertest + MagnumCI
MEAN.js, MEAN.io folosesc Mocha + Karma

# Javascript test runner:

## Chutzpah (118 stars, upd last 8 days)

- folosit de CloudBI
- suporta Jasmine, Mocha, Qunit etc
-se bazeaza pe PhantomJS
- permite rularea testelor din command-line sau din Visual Studio
- dezav: nu are suport pt. grunt/gulp
- se  bazeaza pe blanket.js pt. code coverag (vezi poza asta sau  CloudBI)
- speed mai slab decat la Karma; vezi aici
- tutoriale: aici

## Karma (4919 stars, 12 days)

- creat de Angular team
- Este un command line tool
- poate rula testele folosind un browser real (Chrome, IE, FF, Opera) sau un headless browser (PhantomJS, SlimerJS)
- suporta Jasmine, Mocha, Qunit etc
- se bazeaza pe Istambul pt. code coverage
- integrat cu Jenkis, Travis si Semphore
- dezav: slaba integrare cu VS
- tutoriale: aici

## Comparatii Chtzpah vs. Karma:

- Articol1: => Karma are o mai proasta integrare cu VS (ex: tb. sa rulezi tesele in consola, nu are un plugin pt. Test Explorer). UPDATE: exista un Test Explorer plugin creat de Danny Tuppeny, dar si acesta ruleaza o consola in spate.
- Articol2 (03.2015).
  - Autorul alege Karma + Jasmine + PhantomJs
  - Nu alege Chutzpah pt. ca "No gulp/grunt integration. No SlimerJS support. No remote device support"
- articol3 (12.2014) - "Chutzpah is a great option when you cannot leverage a NodeJS-based tool-chain like Grunt or Gulp, or some other non-Visual Studio build process."
