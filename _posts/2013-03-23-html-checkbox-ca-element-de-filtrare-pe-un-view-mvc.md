---
layout: post
title: "HTML CheckBox - ca element de filtrare pe un View MVC"
date: 2013-03-23 00:00:06
comments: true
categories: MVC javascript
---

Am ajuns sa scriu acest articol dupa ce am incercat sa rezolv o problema care nu ar fi trebuit sa-mi ia mai mult de 15 minute. Sau cel putin asa am crezut!

## Situatia initiala:

Am o **Lista de Produse** si o **Zona de Filtrare** prin care pot selecta doar un subset din totalul produselor. Controalele HTML din zona de filtrare sunt grupate intr-un FORM si sunt transmise la server cu **method="get"**. Valorile din filtrele completate sunt serializate sub forma unui QueryString, direct in URL. Daca nu modific niciun filtru, URL-ul va fi 'curat', ca la prima lansare. Totul functioneaza perfect:

![](/assets/images/2013/catalog1.png)

## Si acum problema:

Am vrut sa adaug in zona de filtrare inca un control de tip CheckBox, prin care sa pot afisa doar produsele aflate pe stoc. Prima ideie, bineinteles, a fost sa folosesc HTML helper-ul din MVC:

```html
@Html.CheckBox("fStock",(bool)ViewBag.StockOnly) //ViewBag - o valoare returnata
de server, utila pentru memorarea starii
```

Aici deja apare prima ciudatenie:

- daca nu bifez checkbox-ul si fac SUBMIT, in URL apare fStock=false
- daca bifez checkbox-ul, in URL apar 2 valori: fStock=true&fStock=false (???...vezi figura)

![](/assets/images/2013/catalog2.png)

Nu, nu am gresit nimic, asa functioneaza helper-ul din MVC. Iar cele 2 valori apar datorita faptului ca, la randare, helper-ul adauga inca un element `<input type="hidden">`. Detalii [aici](http://stackoverflow.com/questions/2697299/asp-net-mvc-why-is-html-checkbox-generating-an-additional-hidden-input).

Chiar si asa, problema nu e greu de rezolvat:

## Solutia 1:

Pe server, citesc parametrii din querystring (eu folosesc un Action Filter descries de ungood aici). In functie de acestia, returnez in ViewBag valoare corespunzatoare (true sau false) a.i, atunci cand View-ul se reincarca, in checkbox sa ramana optiunea setata de utilizator. Codul C# care face acest lucru e urmatorul:

```csharp
//Filter by StockOnly (pentru @HTML.CheckBox)
var fStock = (querystring["fStock"]); //returneaza "true,false" pt. @Html.CheckBox bifat
if (fStock != null && bool.Parse(fStock.Split(',')[0]))
     ViewBag.StockOnly = true;
else
     ViewBag.StockOnly = false;
```

Acum totul functioneaza dar mai am o problema de ordin estetic. Nu-mi place sa vad in URL doua valori diferite pentru Checkbox, atunci cand eu am nevoie doar de una. Si mai ales, nu-mi place sa vad in URL un parametru legat de Stoc, atunci cand eu nu activez acest filtru. Cel mai bine se vede acest lucru atunci cand niciun filtru nu e completat:

![](/assets/images/2013/catalog3.png)

## Solutia 2:

Pot corecta cele de mai sus folosind un control HTML clasic `<input type='checkbox'>`. Ma voi baza pe proprietatea acestuia de a nu transmite nicio valoare atunci cand nu este selectat. In acest caz, codul de pe client, respective server, se modifica astfel:

```html
<input id="fStock" type="checkbox" value="true"  name="fStock"
     @if ((bool)ViewBag.StockOnly) {<text>checked="checked"</text>} />
```

iar pe server:

```csharp
//Filter by StockOnly (pentru <input type="checkbox">)
var fStock = (querystring["fStock"]);
if (!string.IsNullOrEmpty(fStock))
	ViewBag.StockOnly = true;
else
	ViewBag.StockOnly = false;
```

In felul acesta, voi avea un singur parametru in URL (daca bifa e pusa) si niciun parametru (daca bifa lipseste). Adica exact ceea ce am dorit:

![](/assets/images/2013/catalog4.png)

## Solutia 3 (nerecomandata)

Pot trimite FORM-ul ce contine campurile de filtrare folosind atributul **method="post"**. Desigur, e nenatural sa folosesc un POST pentru o cerere care nu altereaza nicio informatie de pe server, doar aduce date. Tot ce am scris mai sus este valabil si aici doar ca valorile campurilor vor fi transmise in BODY-ul pachetului HTTP si nu ca parametri in URL.

Pe client pot folosi acelasi helper MVC:

```html
@Html.CheckBox("fStock",(bool)ViewBag.StockOnly) //ViewBag - o valoare returnata
de server, utila pentru memorarea starii
```

iar pe server codul care actualizeaza ViewBag-ul se modifica doar prin modul de citire a parametrilor:

```csharp
var fStock = Request.Form.GetValues("fStock"); //returneaza "true,false" pt. @Html.CheckBox bifat
if (fStock != null && bool.Parse(fStock[0]))
	ViewBag.StockOnly = true;
else
	ViewBag.StockOnly = false;
```

Inca odata, aceasta ultima solutie am prezentat-o doar cu scop informativ.

## Alte informatii despre `<input type='checkbox'>`:

- se considera 'bifat' daca contine atributul `checked='checked'` si se considera debifat daca acest atribut (si implicit valoarea acestuia) lipseste.
- daca nu e bifat, la SUBMIT browser-ul nu va transmite nici o valoare pentru acest control. Ca si cand campul nu ar exista pe FORM
- daca e bifat, browser-ul va trimite valoarea din atributul 'value'. In cazul de mai sus am folosit value=true dar in loc de true puteam pune orice valoare
- daca e bifat si atributul 'value' lipseste (implicit si valoarea acestuia), atunci se va trimite valoarea '**on**' (??? – alta ciudatenie)
- daca e bifat si atributul exista dar nu are setata o valoare (ex: 'value=') atunci nu se transmite nimic (ca si cand nu ar fi bifat)

## Considerente finale:

- Daca checkbox-ul corespunde unui data **Model** ce urmeaza a fi trimis spre memorare pe server (ex: aplicatie de tip CRUD), atunci **foloseste POST**. Binding-ul se face automat, nu ai parametri in URL si nu mai ai nevoie de ViewBag. Poti folosi in acest sens **@Html.EditorFor**
- Daca checkbox-ul nu apartine unui data Model ci este folosit doar pe post de parametru de filtrare, atunci **foloseste GET**. Vei putea astfel beneficia de mecanismele de caching, URL-ul ce contine filtrarea personalizata ar putea fi transmis la alte persoane sau memorat in favorites.
