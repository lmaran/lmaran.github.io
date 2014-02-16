---
layout: post
title:  "Afisarea raspunsului oferit de Web API in JSON in loc de XML"
date:   2014-02-15 00:00:01
comments: true
categories: Infrastructure
---

## Descrierea problemei ##

Sa definesc intai ce problema vreau sa rezolv. Pentru aceasta am sa creez o aplicatie ASP.NET WebApi cu setarile default:

 ![](https://dl.dropboxusercontent.com/u/43065769/blog/images/2014/default-webapi-first-page.png)

Fara nicio alta setare suplimentara, lansez aplicatia in executie si accesez endpoint-ul: `/api/values` de pe Chrome, FF si IE. 
Sub fiecare poza am atasat:

- valoarea atributului **Accept** pe care browser-ul o ataseaza  fiecarui request
- valoarea atributului **Content-Type** prin care server-ul informeaza browser-ul in ce format ii va furniza raspunsul

### Chrome ###

 ![](https://dl.dropboxusercontent.com/u/43065769/blog/images/2014/chrome-api-values-result.png)

Chrome headers:

- Request: `Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,*/*;q=0.8` 
- Response: `Content-Type: application/xml; charset=utf-8`

### Firefox ###

 ![](https://dl.dropboxusercontent.com/u/43065769/blog/images/2014/ff-api-values-result.png)

FF headers:

- Request: `Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8`
- Response: `Content-Type: application/xml; charset=utf-8`

### IE ###

 ![](https://dl.dropboxusercontent.com/u/43065769/blog/images/2014/ie-api-values-result.png)

IE request header:

- Request: `Accept: text/html, application/xhtml+xml, */*`
- Response: `Content-Type: application/json; charset=utf-8`

De aici se nasc cateva **intrebari**:

- De ce Chrome si FF afiseaza raspunsul in XML si IE in JSON?
- De ce IE iti returneaza raspunsul in JSON dar iti propune sa-l descarci sub forma de fisier?
- Cum poti face ca toate browserele sa-ti afiseze un JSON? Poti face asta chiar si fara sa "atingi" codul?

La toate aceste intrebari am sa incerc sa raspund mai jos.

## Identificarea cauzei ##

**Implicit**, serviciul WebApi stie sa furnizeze raspunsul in ambele tipuri de formate (XML, JSON). La primirea unui request, acesta inspecteaza proprietatea **Accept** din header-ul HTTP (vezi valoarea de sub fiecare poza) si ii raspunde clientului (browser-ului) in functie de "preferinta" setata in acest atribut:

- in cazul Chrome si FF, campul **Accept** contine valoarea `application/xml`, asadar, server-ul nu face decat sa se conformeze: returneaza un XML
- in lista de "preferinte" trimise de IE nu figureaza nici JSON, nici XML, prin urmare server-ul raspunde cu formatarea default: adica JSON

**Obs**: In cazul de fata, atributul **Content-Type** este este atributul prin care server-ul comunica clientului tipul datelor returnate. Acelasi atribut poate fi folosit si de catre client, in acelasi scop, atunci cand are date de transmis server-ului (POST). Alte detalii despre `Accept` si `Content-Type` am notat [aici](http://maran.ro/2013/11/10/github-pe-post-de-cdn-pentru-fisiere-statice-js/).

## Solutia 0: Adauga in Windows registry un MIME Type corespunzator (doar pt. IE) ##

Am notat-o ca fiind "solutia 0" fiindca, de fapt, IE primeste deja datele in format JSON. Deci aici nu avem o problema cu "comutarea" din XML in JSON ci mai degraba cu afisarea acestor date pe ecran.

IE se bazeaza pe MIME Type-urile din Windows si cum Windows-ul nu contine implicit un MIME Type pt. JSON, IE-ul **nu** randeaza raspunsul primit ci lasa aceasta sarcina pe seama utilizatorului (propunandu-i un download).

Partea buna este ca in Windows poti adauga propriile MIME Type-uri. Acest lucru se poate face adaugand doua chei in registry. O poti face manual dar cea mai simpla varianta este urmatoarea (detalii [aici](http://blogs.bullinnovations.com/how-to-enable-json-view-in-internet-explorer/)):

1. Copiaza textul de mai jos intr-un fisier de tip text cu extensia `.reg`

	```batch
	[HKEY_CLASSES_ROOT\MIME\Database\Content Type\application/json]
	"CLSID"="{25336920-03F9-11cf-8FD0-00AA00686F13}"
	"Encoding"=hex:08,00,00,00
	
	[HKEY_CLASSES_ROOT\MIME\Database\Content Type\text/json]
	"CLSID"="{25336920-03F9-11cf-8FD0-00AA00686F13}"
	"Encoding"=hex:08,00,00,00
	```

2. Executa (dublu-click) fisierul de mai sus.