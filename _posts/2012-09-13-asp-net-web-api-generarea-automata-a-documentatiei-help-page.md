---
layout: post
title: "ASP.NET Web Api - Generarea automata a documentatiei (Help Page)"
date: 2012-09-13 00:00:01
comments: true
categories: WebAPI
---

Este vorba de [o facilitate](http://www.nuget.org/packages/Microsoft.AspNet.WebApi.HelpPage) care genereaza documentatia automat pentru resursele REST API, pe baza comentariilor cu care este "ornamentat" codul.

Exemplu de cod:

![](/assets/images/2012/CodeComment.png)

...si documentatia generata:

![](/assets/images/2012/HelpDemo.png)

## Cum se face asta?

Recomand intai parcurgerea acestui [video tutorial](http://blogs.msdn.com/b/yaohuang1/archive/2012/08/15/introducing-the-asp-net-web-api-help-page-preview.aspx). Pe scurt, pasii sunt urmatorii:

1. Din Visual Studio, instalezi [ASP.NET Web Api Help Page](http://www.nuget.org/packages/Microsoft.AspNet.WebApi.HelpPage)

2. Configurezi proiectul sa genereze documentatia intr-un fisier XML:

![](/assets/images/2012/ConfigDocXml2.png)

Astfel, fisierul cu documentatia in format .xml se genereaza automat, la fiecare compilare.

**Observatie**: Odata facuta aceasta activare, compilatorul genereaza un Warning pentru fiecare clasa sau metoda publica care nu este comentata corespunzator. Avertismentul ar fi util daca s-ar aplica doar la codul propriu, dar se aplica si la toate clasele incluse in bibliotecile externe la care faci referire. La mine, de exemplu, dupa ce am activat generarea automata a documentatiei s-au generat cateva sute de warning-uri, fiindu-mi greu astfel sa decelez intre alertele “utile” si cele de tip "lipsa comment":

![](/assets/images/2012/MultipleWarnings.png)

O solutie ar fi folosirea directivei [#pragma warning](<http://msdn.microsoft.com/en-us/library/441722ys(v=vs.80).aspx>) in interiorul fiecarei clase. Dezavantaje:

- volumul mare de munca
- nu poti face acest lucru in clasele aflate in biblioteci externe

O alta solutie (asta am folosit eu) ar fi adaugarea valorii **1591** in sectiunea "**Supress warning**", ca in figura prezentata putin mai sus. In acest fel, sunt filtrate erorile de tip "lipsa comentariu" si raman doar atentionarile utile:

![](/assets/images/2012/2warnings.png)

3. Actualizeaza in Areas/HelpPage/App_Start/HelpPageConfig.cs adresa fisierului ce contine documentatia .xml:

![](/assets/images/2012/adrxml.png)

4. Optional, modifica layout-ul paginii de help a.i. sa fie in concordanta cu layout-ul intregii aplicatii MVC:

![](/assets/images/2012/layout.png)

5. Optional, poti decora un anumit controller sau doar o metoda cu atributul `[ApiExplorerSetting(IgnoreApi=true)]` a.i. acestea sa fie excluse din procesul de generare a documentatiei. Sau poti face acest lucru prin cod, asa cum este prezentat [aici](http://blogs.msdn.com/b/kiranchalla/archive/2012/09/04/opting-in-controllers-to-show-up-on-help-page.aspx).

6. Modulul "HelpPage" determina automat denumirea si tipul proprietatilor pentru un anumit tip de date, si le initiaza cu niste valori predefinite:

![](/assets/images/2012/defaultvalues.png)

Daca doresti, poti folosi fisierul _HelpPageConfig.cs_ pt. a-ti define propriile exemple (valori) pt. un anumit tip, astfel:

![](/assets/images/2012/customvalues.png)

## Rezultat:

![](/assets/images/2012/customvalues2.png)

Unele actiuni pot returna **HttpResponseMessage** in loc de un tip explicit definit. Pagina de Help nu recunoaste acest tip si, in acest caz, nu afiseaza nimic in sectiunea de "Response". Solutia consta tot intr-o modificare in fisierul _HelpPageConfig.cs_:

![](/assets/images/2012/config.png)

Astfel, pentru o metoda API care returneaza un **HttpResponseMessage** ca mai jos:

![](/assets/images/2012/postmethod.png)

...se va genera o documentatie a carei sectiune de tip "Response" va arata astfel:

![](/assets/images/2012/response.png)

...unde tipul "_CrmInvoiceDetail_" putea fi si el customizat dupe metoda prezentata mai sus, la "Counter".
