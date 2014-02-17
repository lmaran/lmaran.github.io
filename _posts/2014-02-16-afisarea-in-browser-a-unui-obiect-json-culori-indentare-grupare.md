---
layout: post
title:  "Afisarea in browser a unui obiect JSON: culori, indentare, grupare"
date:   2014-02-16 00:00:01
comments: true
categories: Infrastructure
---

## Definirea problemei ##

Atunci cand faci debug si lucrezi cu servicii REST simple (GET, fara autentificare) nu ai nevoie de tool-uri specializate (fiddler, postman etc). E suficient un browser, dar pt. o vizualizare optima a rezultatuilui ai nevoie de cateva ajustari:

- in primul rand, mai mult ca sigur vei dori ca rezultatul sa fie afisat in format JSON (in loc de XML). Detalii despre acest lucru am scris in [articolul precedent](http://maran.ro/2014/02/15/afisarea-raspunsului-returnat-de-webapi-in-json-in-loc-de-xml/).
- in al 2-lea rand, vei dori ca JSON-ul returnat sa fie afisat intr-o forma cat mai lizibila, iar descrierea acestui lucru reprezinta scopul prezentului articol.

Am modificat controller-ul "Values" dintr-un proiect WebAPI default a.i. sa returneze doua obiecte ceva mai complexe. Rezultatul in cele 3 browser-e (Chrome, FF, IE) este urmatorul:

![](https://dl.dropboxusercontent.com/u/43065769/blog/images/2014/plain-json-chrome-ff-ie.png)

Se observa ca rezultatul este interpretat ca un simplu text (string), fara nicio formatare. Putem face mai lizibil acest text configurand corespunzator  server-ul, browser-ul sau ambele.

### Solutia 1. Formatarea rezultatului prin configurarea server-ului ###

Ce **pot suporta** prin aceasta metoda:

- indentare
- camelCase (util daca datele urmeaza a fi consumate in Javascript)

Ce **nu pot suporta** - in general tot ce tine de styling:

- culori (culori diferite in functie de tipul datelor)
- hyperlinks
- grupari (expand/colapse)

Configurare codului (ma refer la Web API) se face adaugand in fisierul de configurare urmatoarele linii:

```csharp
var jsonFormatter = config.Formatters.JsonFormatter;
var settings = jsonFormatter.SerializerSettings;
settings.Formatting = Newtonsoft.Json.Formatting.Indented; // Indenting
settings.ContractResolver = new CamelCasePropertyNamesContractResolver(); // Camel Casing
```

Chrome si FF vor fi happy cu aceste setari:

![](https://dl.dropboxusercontent.com/u/43065769/blog/images/2014/webapi-json-formatter-chrome.png)

![](https://dl.dropboxusercontent.com/u/43065769/blog/images/2014/webapi-json-formatter-ff.png)

Din pacate, pentru IE au efect doar setarile de camelCase:

![](https://dl.dropboxusercontent.com/u/43065769/blog/images/2014/webapi-json-formatter-ie.png)

## Solutia 2. Formatarea rezultatului din browser ##

### IE ###

- pentru IE nu stiu niciun plugin care sa poata fi folosit pt. formatarea JSON-ului. Poate tu?

### Chrome ###

- instaleaza extensia [JsonView](https://chrome.google.com/webstore/detail/jsonview/chklaanhfefbnpoihckbnefhakgolnmc), iar rezultatul va arata asa:

 ![](https://dl.dropboxusercontent.com/u/43065769/blog/images/2014/webapi-json-formatter-chrome-nice.png)

### FF ###

- instaleaza aceeasi extensie [JsonView](https://addons.mozilla.org/en-us/firefox/addon/jsonview/) (varianta FF), iar rezultatul va arata identic ca in Chrome:

 ![](https://dl.dropboxusercontent.com/u/43065769/blog/images/2014/webapi-json-formatter-ff-nice.png)

Obs:

- extensia *JsonView* **nu** realizeaza si transformarea in camelCase. Aceasta apare in cele doua imagini pt. ca am pastrat setarile facute pe server (vezi solutia 1)
- extensia *JsonView* realizeaza si indentarea, indiferent daca ai specificat, sau nu, acest lucru in codul WebApi