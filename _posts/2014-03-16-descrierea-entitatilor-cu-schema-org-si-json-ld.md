---
layout: post
title:  "Descrierea entitatilor cu Schema.org si JSON-LD"
date:   2014-03-16 00:00:01
comments: true
categories: software infrastructure
---

## Schema.org ##

[Schema.org](http://schema.org) este un site ce contine o colectie de descrieri (schema) a celor mai uzuale entitati folosite in aplicatiile web (Person, Product, Movie etc). De aceasta standardizare beneficiaza in primul rand motoarele de cautare, motiv pt. care **Google**, **Bing** si **Yahoo** se regasesc printre sustinatorii acestui proiect.

Concret, implementarea acestui standard se bazeaza pe tag-urile de tip [Microdata](http://dev.w3.org/html5/md-LC/) introduse odata cu HTML5:

- `itemscope`
- `itemtype`
- `itemprop`

## Exemplu ##

Sa consideram aceasta descriere HTML:

```
<div>
 <h1>Avatar</h1>
 <span>Director: James Cameron (born August 16, 1954)</span>
 <span>Science fiction</span>
 <a href="../movies/avatar-theatrical-trailer.html">Trailer</a>
</div>
```
Oricine poate recunoaste cu usurinta faptul ca:

- sectiunea de mai sus se refera la un *film* (nu la o *poza de profil*)
- *Avatar* este numele filmului
- *James Cameron* este numele regizorului
- *Science fiction* reprezinta genul filmului

Toate cele de mai sus par evidente atunci cand sunt citite de catre o persoana, nu si atunci cand sunt interpretate de o masina (search engine). 

Acelasi exemplu, rescris  pe bazeaza pe baza descrierii de la adresa [http://schema.org/Movie](http://schema.org/Movie) devine:

```
<div itemscope itemtype ="http://schema.org/Movie">
  <h1 itemprop="name">Avatar</h1>
  <span>Director: <span itemprop="director">James Cameron</span> (born August 16, 1954)</span>
  <span itemprop="genre">Science fiction</span>
  <a href="../movies/avatar-theatrical-trailer.html" itemprop="trailer">Trailer</a>
</div>
```

De aceasta data, atat omul cat si masina pot recunoaste cu exactitate atat sensul unor anumite informatii (ex: nume, gen) cat si contextul din care acestea provin (ex: film)

Mai multe detalii despre acest exemplu se regasesc [aici](http://schema.org/docs/gs.html#microdata_why).

## JSON-LD ##

OK, acum stim cum putem obtine beneficii maxime (SEO) de la motoarele de cautare: stim ca pe langa datele utile, mai avem nevoie de niste descrieri ale respectivelor date (asa numitele `metadata`). Si cum majoritatea aplicatiilor moderne ambaleaza si transmit datele de la server catre client in format JSON, solutia naturala este ca si metadatele sa fie transportate cu acelasi "vehicul".

[JSON-LD](http://www.w3.org/TR/json-ld-syntax/) (Linked Data) este un standard care [se foloseste](http://blog.schema.org/2013/06/schemaorg-and-json-ld.html) de vocabularul definit la schema.org pentru a descrie diverse entitati (data + metadata) in format JSON.

Practic, JSON-LD este de fapt un JSON standard, cu cateva restrictii:

- cheile sunt unice
- o parte din chei reprezinta cuvinte cheie (ex: @context, @type). 
	- Valorile pe care le poate lua param @type sunt specificate la [schema.org](http://schema.org/docs/full.html)
- numele celorlate chei (proprietati) trebuie sa fie in acord cu modul in care este descris tipul respectiv pe schema.org. De exemlu, pt. @type=`Movie` vor fi permise, printre altele atributele: `name`, `director` si `genre`.
 
## Exemplu ##

Descrierea HTML de mai sus, transpusa in format JSON-LD, devine:

````
{
	"@context" : "http://schema.org",
	"@type" : "Movie",
	"name" : "Avatar",
	"director" : "James Cameron",
	"genre" : "Science fiction"
}
````

## JSON IntelliSese bazat pe JSON-LD si Schema.org##

Mad Kristensen (autorul lui Web Essentials) prezinta in acest [video](http://www.youtube.com/watch?v=dwURmZ71sj8) suportul IntelliSense in JSON pentru entitatile descrise in schema.org

![](https://dl.dropboxusercontent.com/u/43065769/blog/images/2014/json-intellisense-in-vs-with-schema-org-and-json-ld.png)

- se observa ca proprietatile `name`, `address` sau `streetAddress` sunt valori predefinite pe site-ul schema.org pentru tipul [Person](http://schema.org/Person). 

## Considerente de final ##

[Good Relations](http://www.heppnetz.de/projects/goodrelations/) este un alt proiect (mai vechi) care [ofera](http://wiki.goodrelations-vocabulary.org/Documentation/Intro) aplicatiilor de tip e-commerce posibilitatea de a-si descrie entitatile folosind un vocabular unic . Conform propriul lor [blog](http://blog.schema.org/2012/11/good-relations-and-schemaorg.html), schema.org integreaza acum si structurile de date ale Good Relations.

Acum stiu ca, in momentul in care voi scrie urmatoarea aplicatie, voi incerca, pe cat posibil, sa refolosesc [descrierea tipurilor](http://schema.org/docs/full.html) de pe schema.org.