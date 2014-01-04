---
layout: post
title:  "ASP.NET Web Api - apeluri RESTfull si RPC pentru aceeasi resursa"
date:   2012-09-16 00:00:01
comments: true
categories: WebApi
---

In mod standard, ASP.NET Web Api nu permite actiuni multiple, cu aceeasi *semnatura*, catre aceeasi resursa (Controller). Altfel, returneaza eroarea: "**Multiple actions were found that match the request...**".

In contextul WebApi, doua metode au aceeasi *semnatura* daca:

- corespund aceluias **verb** (GET, POST etc) si
- au acelasi **numar** de parametrii si
- **numele** parametrilor este identic

Atentie, nu conteaza **tipul** returnat sau **tipul** parametrilor!

Bine, dar concret, in ce situatie as avea nevoie de asa ceva?

## Exemplu (problema): ##

Vreau sa pun la dispozitie 2 API-uri care raspund la GET si care au un singur parametru:

- Api1 – returneaza un produs pe baza ID-ului acestuia (ID=Int)
- Api2 – returneaza un produs pe baza Codului acestuia (Cod=String)

## Solutia: ##

Pentru Api1 voi folosi ruta default iar pentru Api2 voi adauga o noua ruta de tip "RPC":

```csharp
config.Routes.MapHttpRoute(
   name: "RpcApi",
   routeTemplate: "v1/rpc/{controller}/{action}/{id2}" //obligatoriu, numele param. tb. sa fie diferit
);

config.Routes.MapHttpRoute(
   name: "DefaultApi",
   routeTemplate: "v1/{controller}/{id}",
   defaults: new { id = RouteParameter.Optional }
);
```

iar in Controller:


```csharp
public string GetByCode(string id2) //RPC
{
    return "value by code (RPC call)";
}

public string Get(int id) //RESTfull
{
    return "value by id (RESTfull call)";
}
```

## Rezultatul: ##

Apel RESTfull:

![](https://dl.dropboxusercontent.com/u/43065769/blog/images/2012/Restfull.png)

Apel RPC:

![](https://dl.dropboxusercontent.com/u/43065769/blog/images/2012/Rpc.png)


## Discutii pe marginea solutiei: ##

- Operatiile elementare de tip CRUD se fac cu apelurile standard (RESTfull). Daca insa este nevoie de adaugarea uneia (sau a mai multor) actiuni suplimentare, atunci acestea se vor apela prin URL-uri de tip RPC;
- Daca numarul parametrilor coincide (vorbim de metode RPC si RESTfull pt. acelasi verb si aceeasi resursa), atunci **numele parametrilor trebuie sa difere** (id vs. id2);
- Nu conteaza ordinea in care cele doua rute sunt declarate;
- Exceptand URL-ul prin care se activeaza, actiunile RPC nu difera cu nimic de actiunile standard (RESTfull). Ramane valabila, asadar, regula de mapare a verbelor:
	- prin **conventie** de nume (inceputul numelui sa contina numele verbului);
	- sau **explicit** (prin decorarea metodei cu un atribut corespunzator);
- Solutia prezentata functioneaza pentru toate verbele Http (GET, POST, PUT, DELETE);
- Ca si **dezavantaj**, nu se pastreaza o consistenta in formatul URL-urilor folosite in cele doua situatii.

In concluzie, solutia prezentata mai sus patreaza nealterata operarea cu resurse in modul standard (RESTfull) dar ofera si o modalitate pentru introducerea de operatii suplimentare.

## Solutii alternative: ##

### 1. Rutare pe baza de actiune (RPC 100%) ###

**Avantaje:**

- necesita o singura ruta de tip RPC ("api/{controller}/{action}/{id}")
- nu ai probleme cu numarul sau numele parametrilor
- un singur pattern pentru URL (consistenta)

**Dezavantaje:**

- renunti complet la simplitatea apelurile de tip RESTfull  (bazate doar pe "substantiv"):
	- aici ar fi o exceptie…in definitia rutei poti folosi o actiune default care sa se execute (Ex: Index() sau Get() sau...) in caz ca numele actiunii lipseste (Ex: "/products"). Din pacate, daca trebuie sa mai folosesti un POST, atunci va trebui sa specifici si numele actiunii (Ex: "/products/create").

### 2. Controller-e multiple pentru aceeasi resursa (RESTfull 100%) ###

**Avantaje:**

- necesita o singura ruta de tip RESTfull (“api/{controller}/{id}”)
- nu ai probleme cu numarul sau numele parametrilor
- un singur pattern pentru URL (consistenta)

**Dezavantaje:**

- altereaza si multiplica numele resursei. Folosind exemplul prezentat anterior,  URL-urile pt. cele 2 situatii ar fi:
	- "/products/1"
	- "/productsbycode/code1", sau orice alt nume diferit de "products"

## Referinte: ##

[REST vs. RPC in ASP.NET Web API? Who cares; it does both.](http://encosia.com/rest-vs-rpc-in-asp-net-web-api-who-cares-it-does-both/) (Dave Ward, Encosia blog, 04.2012)

[An Introduction to ASP.NET Web API partea de rutare](http://weblog.west-wind.com/posts/2012/Aug/21/An-Introduction-to-ASPNET-Web-API) (Rich Strahl, 08.2012)

[Passing multiple POST parameters to Web API Controller Methods](http://weblog.west-wind.com/posts/2012/May/08/Passing-multiple-POST-parameters-to-Web-API-Controller-Methods), prima parte (Rich Strahl, 05.2012)

[Routing and Action Selection](http://www.asp.net/web-api/overview/web-api-routing-and-actions/routing-and-action-selection) (Mike Wasson, 07.2012)

[Opinionated (RPC) APIs vs RESTful APIs](http://blog.perfectapi.com/2012/opinionated-rpc-apis-vs-restful-apis/) (03.2012)