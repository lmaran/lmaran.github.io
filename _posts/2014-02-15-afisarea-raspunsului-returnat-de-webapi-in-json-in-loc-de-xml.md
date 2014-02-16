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
