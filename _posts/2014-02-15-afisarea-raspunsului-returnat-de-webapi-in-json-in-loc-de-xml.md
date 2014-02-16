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

In acest moment **IE** stie ca toate raspunsurile care contin in atributul **Content-Type** valoarea `application/json` sau `text/json` le poate randa ca pe orice text obisnuit:

 ![](https://dl.dropboxusercontent.com/u/43065769/blog/images/2014/ie-api-values-result-as-json.png)

## Solutia 1: Modifica codul WebAPI a.i. sa returneze JSON in loc de XML ##

Asa cum am vazut mai sus, pentru **IE** nu trebuie sa modific nimic.  Fie ca este descarcat intr-un fisier, fie ca este afisat in browser (cu un MIME type configurat in prealabil), raspunsul primit de IE de la WebApi este JSON. Asta pt. ca **WebApi returneaza JSON by default** si XML doar daca i se cere explicit.

Prin atributul **Accept**, un browser poate sa ceara ca datele sa-i fie livrate intr-un anumit format, dar server-ul poate sa ignore aceasta cerere/propunere si sa returneze datele in ce format doreste. Asadar, pt. **Chrome** si **FF** putem folosi una din cele 2 abordari:

### 1.1. Dezactivarea "XML Formatter"-ului din aplicatia WebApi. ###
 
Odata serializatorul de XML eliminat, server-ul nu mai are alternativa si va raspunde cu singurul serializator disponibil (JSON). Pot face asta in mai multe moduri:

- sterg direct XML formatter-ul (detalii [aici](http://www.asp.net/web-api/overview/formats-and-model-binding/json-and-xml-serialization#removing_the_json_or_xml_formatter)):

 ```csharp
 config.Formatters.Remove(config.Formatters.XmlFormatter);
 ```
- sterg toate formatter-ele si adaug doar ce ma intereseaza (detalii [aici](http://stackoverflow.com/a/20192316)):

 ```csharp
 config.Formatters.Clear();
 config.Formatters.Add(new XmlMediaTypeFormatter());
 config.Formatters.Add(new JsonMediaTypeFormatter());
 config.Formatters.Add(new FormUrlEncodedMediaTypeFormatter());
 ```
 
 Observatii:
 - formattere-le sunt procesate in ordinea in care sunt adaugate si se activeaza doar prima care se potriveste
 - adaugand XML inainte de JSON pot face ca IE-ul sa primeasca un XML
- elimin cu totul procesul prin care se "negociaza" un anumit formatter (este solutia **cea mai performanta**, detalii in [articolul lui Fillip W.](http://www.strathweb.com/2013/06/supporting-only-json-in-asp-net-web-api-the-right-way/))

### 1.2. Selectarea unui anumit Formatter (JSON/XML) din query string ###

- este o varianta pe care personal nu o agreez. Exemplu de folosire (detalii [aici](http://stackoverflow.com/a/19978996) sau [aici](http://stackoverflow.com/a/11357371)):

 ```csharp
 for xml : http://localhost:16600/api/values/?type=xml
 for json: http://localhost:16600/api/values?type=json
 ```

- avantajul acestei metode este ca imi permite sa pastrez ambele formate