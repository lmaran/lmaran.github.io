---
layout: post
title:  "Alternative la Jekyll"
date:   2023-07-02 21:19:24 +0300
categories: Static site generator
---
[Gitbook](https://www.gitbook.com/pricing#pricingTable), [MdBook](https://rust-lang.github.io/mdBook/) și [Docusaurus](https://docusaurus.io/) sunt printre cele mai bune alternative la Jekyll.

# Gitbook

- gratuit (1 user)
- SaaS (hosting inclus)
- editor (doar) online
  - înainte aveau un cli cu care puteai edita local și sincroniza cu Git
  - util pentru scoala21 - unde secretara poate edita conținutul online (nu necesită github și sincronizare)
  
- intern, salvează conținutul într-o DB și servește conținutul dinamic (nu doar static pagis), deci nu apare .html în link-uri
- nu e lent, dar nici rapid - se simte când treci de la o pagină la alta
- nu am reușit să scap de primul segment din URL, adică adresa arată asa:
  - https://scoala21.gitbook.io/doc/ unde in loc de  'doc' poți pune orice, dar nu-l poți elimina (se presupune că 'scoala21' este organizatia și doc este unul dintre proiectele organizatiei)
  - poți seta gratuit un custom domain - așa dispare și acel segment 'doc' 
- apare o etichetă 'Powered by Gitbook' in colțul sțngă-jos, de care nu poți scăpa.
- search integrat
- doar două teme (dark/light)
- in planul free poți customiza doar paleta primară de culori (nu și tema)
- nu poți elimina de pe fiecare pagină mesajul "last modified 18m ago"

# MdBook

- Scris în Rust, de echipa Rust.
- Documentația din Rust (și multe alte proiecte) folosesc MdBook
- clean design
- Seamănă f. mult cu Jekyll
- se descarcă un cli (scris în Rust) și se editează local, apoi push to git
  - convertește Markdown în Html
  - la build se generează un folder unde se pun fișierele html generate
- foarte rapid (servește pagini statice)
- search integrat
- url-uri cu HTML. Exemplu: https://codegangsta.gitbooks.io/building-web-apps-with-go/content/testing/unit_testing/index.html

# Docusaurus

- gratuit, mulți îl folosesc (45k github stars)
- foarte multe facilități - destul de bine cotat
- pare cam lent (bazat pe React)
- este tot un static site generator (ca și MdBook), dar mai complex

# In loc de concluzie

Cele 3 de mai sus sunt special concepute pentru a găzdui documentație, dar pot fi folosite și ca blog-uri sau site-uri web obișnuite. Din categoria Static site generator mai fac parte Jekyll (Ruby), Hugo (Golang) sau Gatsby(React+NodeJs)
