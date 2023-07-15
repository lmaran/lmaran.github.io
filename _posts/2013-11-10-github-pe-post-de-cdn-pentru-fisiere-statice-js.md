---
layout: post
title: "Github pe post de CDN pentru fisiere statice (js). Probleme si solutii."
date: 2013-11-10 00:00:01
comments: true
categories: Dev-infrastructure
---

Totul a plecat ca urmare a unei descoperiri intamplatoare.

## Problema:

Pregatind un [demo](http://jsfiddle.net/lmaran/XL4S6/15/) pentru unul din articolele precedente, am descoperit ca aplicatia (rulata in jsFiddle) functiona foarte bine in FF, dar nu si in Chrome sau IE. Fiind vorba doar de niste resurse statice (HTML, JS, CSS) am incercat sa copiez intreg codul intr-un singur fisier _.html_ si sa-l rulez independent de contextul jsFiddle. Acelasi rezultat.

Pentru a pune in evidenta aceasta problema ma voi folosi de un exemplu concret: doresc sa folosesc biblioteca [jqTree](http://mbraak.github.io/jqTree/) pentru a afisa un arbore cu 2 noduri.

- **Exemplu**: [http://jsfiddle.net/lmaran/t9N9W/1/](http://jsfiddle.net/lmaran/t9N9W/1/) (varianta **A**)
  - ok pe **FF**(25.0)
  - non ok pe **IE**(10) si **Chrome**(30.0.1599)

## Cauza si solutia:

Dupa scurt timp am identificat cauza: foloseam libraria [jqTree](http://mbraak.github.io/jqTree/) in format 'raw' [apelata direct de pe Github](https://raw.github.com/mbraak/jqTree/master/tree.jquery.js). Am pus [acelasi fisier pe Dropbox](http://dl.dropboxusercontent.com/u/43065769/tree.jquery.js) si a totul a functionat ok, pe toate browser-ele.

- **Exemplu**: [http://jsfiddle.net/lmaran/uL8Jr/](http://jsfiddle.net/lmaran/uL8Jr/) (varianta **B**, full ok)

## Ok, acum ca stiu cum sa rezolv problema, ce mai e?

Curiozitatea. De ce cu biblioteca apelata de pe Github aplicatia nu functioneaza? Care este **adevarata** cauza a problemei (_root cause_)?

In plus, mi-am amintit ca nu o data am intalnit probleme (pe anumite browser-e) cu codul de pe **jsFiddle** la care diverse raspunsuri (Stackoverflow) faceau referire. Ar putea sa existe vreo legatura intre aceste probleme si scenariul descris mai sus? M-am hotarat sa investighez, dar sa o luam pe rand:

- Q1 – de ce varianta **A** nu functioneaza?
- R1 – pentru ca biblioteca jqTree (de pe Github) nu se incarca:

![](/assets/images/2013/jqTree-pending.png)

- Q2 – de ce libraria nu se incarca?
- R2 – pentru ca Github nu doreste asta. Nu doreste sa-si transforme server-ele intr-un [CDN](jqTree-MimeError). Detalii [aici](https://github.com/blog/1482-).

![](/assets/images/2013/jqTree-MimeError.png)

- Q3 - Cum se protejeaza Github?
- R3 - Prin campul **Accept** din Request-Header-ul http, clientul ii transmite server-ului ce tip de fisiere (MIME type) este pregatit sa primeasca. Server-ul poate sa tina sau poate sa nu tina cont de'sugestia' clientului. In cazul de fata, Github raspunde cu un mesaj de tipul "_text/plain_" (campul **Content-Type** din Response-Header) in loc de "_application/javascript_" (asa cu raspunde Dropbox). Nici acest lucru nu l-ar impiedica pe client (browser) sa se descurce cu fisierul primit dar Github pluseaza: trimite in header inca un camp (**X-Content-Type-Options**) cu valoarea `nosniff`. Astfel, server-ul ii impune browser-ului sa nu trateze altfel fisierul primit decat in acord cu tipul specificat in **Content-Type**.

Initial doar IE, iar acum si Chrome, au introdus suport pentru `nosniff`. Acesta este motivul pentru care codul din varianta **A** functioneaza (inca) pe FF.
Daca afisam fisierul direct din browser nu avem nicio problema fiindca acesta este prelucrat conform cu indicatiile Github – adica ca si "_text/plain_":

![](/assets/images/2013/jqTree-HttpHeaders.png)

- Q4. Si daca totusi am nevoie de o anumita librarie (.js) care se afla pe Github?
- R4. Depinde:
  - mediu de **productie**:
    - variantele clasice (CDN, download local etc)
    - [Github Pages](http://pages.github.com/) (Ex: http://mbraak.github.io/jqTree/tree.jquery.js)
  - mediu de **testare** (ex: jsFiddle, jsBin sau custom)
    - Dropbox, Azure Blobs etc
    - [RawGithub.com](http://rawgithub.com/) trebuie doar sa elimini punctul (.) dintre **raw** si **github.com** din url-ul Github original. Exemplu:
      - **Before**: //**raw.github**.com/mbraak/jqTree/master/tree.jquery.js
      - **After**: //**rawgithub.com**/mbraak/jqTree/master/tree.jquery.js

**In concluzie**: renunta la ideia de a folosi Github pe post de CDN!

Asta a fost tot.
