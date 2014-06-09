---
layout: post
title:  "Helios versus System.Web"
date:   2014-06-07 00:00:01
comments: true
categories: software infrastructure
---

Conform specificatiilor OWIN, 4 layere intervin atunci cand rulezi o aplicatie web:

![OwinStack](https://dl.dropboxusercontent.com/u/43065769/blog/images/2014/owin-stack.png)

Am sa insist aici doar asupra primelor doua.

## 1. Host Layer ##

Pentru a gazdui o aplicatie web (WebApi) pe o masina Windows ai practic doua optiuni la dipozitie:

1. IIS
2. SelfHost/OwinHost (o Console Application ce poate rula independent sau ca Windows Service)

Exista cateva scenarii in care SelfHost poate fi o solutie viabila (sau unica solutie). Pentru majoritatea scenariilor insca, IIS este "gazda" ideala. 
De ce prefer IIS?

- Deploy simplu:
	- "one click" deploy din VS sau automat deploy din Github (WebHook). 
	- transmite doar ultimele modificari (folosind WebDeploy)
	- publicare "on-the-fly" - nu necesita oprirea procesului in care ruleaza aplicatia (w3wp.exe / iisexpress.exe)
- Zero downtime
	- IIS accepta modificarile facute in folderul BIN sau in web.config fara a pierde vreun request. Restarteaza AppPool-ul, facand ca primele request-uri sa fie intarziate, dar nu rejectate. La o Console App primesti mesajul "File in use" daca incerci sa modifici un fisier deja incarcat in memorie => pt. deploy trebuie in prealabil sa opresti aplicatia.
- Gzip automat - pt. static / dynamic content
- Altele (le-am trecut separat fiindca, personal, le folosesc rar sau deloc):
	- Windows auth.
	- caching
	- web sockets
	- tool-uri de administrare (ex: pt. certificate)
	- etc

## 2. Server Layer ##

Am stabilit ca folosesc IIS, dar acesta se ocupa, in principal, de managementul proceselor. Web Server-ul este cel care preia cererile HTTP, initiaza pipeline-ul OWIN si returneaza raspunsul furnizat de layer-ele superioare.

Daca ai ales un **host** de tip IIS, poti opta in continuare pt. unul din cele 2 **servere**: System.Web sau Helios.

### 2.1. System.Web ###

- `System.Web` (namespace-ul aferent lui **System.Web.dll**, stocat in GAC)
- `Microsoft.Owin.Host.SysytemWeb` (namespace-ul aferent  lui **Microsoft.Owin.Host.SystemWeb.dll**, stocat in folder-ul `bin` - biblioteca care extinde System.Web implementand specificatiile OWIN) 
- ruleaza doar pe un host IIS
- sistem monolitic (all in one), in care aproape toate modulele sunt activate implicit:
	- Http Modules (o parte din aceste module se pot dezactiva din `web.config`)
	- Http Handlers
	- Sessions
	- Cache
	- Web Forms
	- Controls
- faptul ca WebForms se incarca in pipeline-ul HTTP la fiecare request si MVC sau WebApi nu folosesc WebForms => un exemplu de consum inutil de resurse.
- parte integranta din Asp.Net, deci parte componenta al lui .NetFramework. Consecinte:
	- System.Web se actualiza f. rar (~ odata la 2 ani)
	- trebuie sa convingi provider-ul de hosting sa actualizeze .NetFramework-ul daca vrei sa folosesti o versiune mai recenta de System.Web
- ofera un model de programare bazat pe obiectul `HttpContext`
- MVC inca depinde de System.Web (deci de Asp.Net, deci de .NetFramework). WebApi, in schimb, nu mai depinde se System.Web (vezi WebApi in SelfHost). Totusi, pana la aparitia Helios, daca ai fi vrut sa hostezi un WebApi in IIS nu puteai "ocoli" System.Web.

### 2.2. Helios ###

- `Microsoft.Owin.Host.IIS` - disponibil pe Nuget
- ruleaza doar pe un host IIS
- isi propune sa ofere facilitatile IIS fara a fi nevoit sa "platesti" pt. Asp.Net. Ideia este sa poti scrie cod care sa ruleze cu minime interferente din partea layer-elor inferioare.
- nu depinde de System.Web
- tot ce are nevoie aplicatia pt. a rula este in folder-ul `bin` =>  nu mai depinzi  de GAC (deci de hosting provider)
- necesita Windows 8/Server 2012 + Full trust
- necesita minim. .Net 4.5.1.

#### Instalare Helios ####

- instaleaza de pe Nuget pachetul: `Microsoft.Owin.Host.IIS`. Acesta va instala, la randul lui:
	- `Microsoft.Owin.Hosting` (verifica sectiunea 'update' ca sa fi sigur ca ai ultima varianta, altfel primesti eroare)
	- `Microsoft.AspNet.Loader.IIS`
- nu trebuie sa faci nimic altceva (ex: nu tb. sa dezinstalezi in prealabil System.Web)

#### Initializare Helios ####

- in Asp.Net din .Net 4.5.1 s-a introdus logica care "se uita" in folder-ul `bin` dupa fisierul `Microsoft.AspNet.Loader.IIS.dll`. Daca il gaseste, ii preda controlul. In continuare, acest "loader" incarca Helios (in loc de System.Web).
- Helios va fi cel care va sti, in continuare, sa initializeze pipeline-ul Http.

#### Dezinstalare Helios ####

- dezinstalezi, din Nuget, pachetul: `Microsoft.Owin.Host.IIS`. Nimic mai mult. Odata cu el se va dezinstala si `Microsoft.AspNet.Loader.IIS` asa ca, daca inainte ai avut  instalat System.Web, el va fi cel care va prelua in continuare controlul.


## 3. Masurarea performantelor: Helios vs. System.Web ##

Pt. masurarea performantelor am creat creat o aplicatie simpla de tip Owin plecand de la un "Empty" Asp.Net project. Tot codul se afla in fisierul Startup.cs:

```
using Owin;

namespace Helios
{
    public class Startup
    {
        public void Configuration(IAppBuilder app)
        {
            app.Run(async context =>
            {
                context.Response.ContentType = "text/plain";
                await context.Response.WriteAsync("Hello World!");
            });

        }
    }
}
```

Alte considerente legate de conditiile de test:

- CPU i5/2.4 GHz, RAM 4GB, SSD, Windows 8.1, Local IIS (nu IIS Express fiindca nu am putut sa masor req/sec pt. IISExpress din PerfMon)
- Windows Defender dezactivat
- Release mode
- inaintea fiecarui test am restartat IIS-ul (clean up memory), iar prima masuratoare am ignorat-o (warming up).

In toate cazurile am folosit acelasi "stressing tool" (Apache Benchmark) si aceiasi parametri:

`ab -n 70000 -c 50 http://pcdadi/` (ultimul `"/"` e obligatoriu) 

Sunt simulate astfel 70.000 requesturi, cu 50 de clienti (thread-uri).

Doar de curiozitate, am adaugat pentru masuratori si codul de tip "Hello World" prezentat pe homepage-ul NodeJS. Am incercat ca rezultatele sa le suprapun a.i. cele 3 solutii sa poata fi cat mai usor comparate:

![](https://dl.dropboxusercontent.com/u/43065769/blog/images/2014/perfmon-all-ab.png)

Nu incercati sa-l intelegeti :-), am sa-l iau pe rand:

### 3.1. Consum de CPU ###

![](https://dl.dropboxusercontent.com/u/43065769/blog/images/2014/perfmon-cpu-ab.png)

Aici Helios si NodeJs se comporta ambele la fel de bine: se descurca doar cu jumatate din puterea de care are nevoie System.Web pentru a face acelasi job.

Am sa anticipez putin: din figura de mai sus se observa ca "turnul" cel mai ingust corespunde lui Helios. Adica, acelasi test s-a finalizat mai rapid in cazul Helios, deci ne vom astepta ca la testul "requests/sec" acesta sa ofere cel mai bun rezultat.

### 3.2. Rata de transfer (req/sec) ###

![](https://dl.dropboxusercontent.com/u/43065769/blog/images/2014/perfmon-req-ab.png)

1. Helios (5700 req/sec) - cel mai bun
2. System.Web (4200 req/sec)
3. NodeJS (3700 req/sec)

Faptul ca aria incadrata de cele 3 "clopote" este cvasi-identica imi ofera un argument in plus in ceea ce priveste acuratetea rezultatelor (in fiecare test nr. de request-uri a fost acelasi - 70.000).

### 3.3. Memoria alocata (doar "manage" memory) ###

![](https://dl.dropboxusercontent.com/u/43065769/blog/images/2014/perfmon-bytes-ab.png)

Se observa ca Helios consuma de aprox 4 ori mai putina memorie, dar sa nu ne amagim. De unde provine aceasta diferenta? - din faptul ca nu mai incarca, si nu mai proceseaza "balastul" indus de pipeline-ul clasic din Asp.Net. O sa vedem asta in figura ce urmeaza. 

Pe masura ce creste in complexitare, aplicatia va tinde sa devina conumatorul de memorie dominant, reducand astfel decalajul afisat mai sus. 

### 3.4. Numarul de biblioteci (dll-uri) incarcate in memorie ###

![](https://dl.dropboxusercontent.com/u/43065769/blog/images/2014/perfmon-asm-ab.png)

Acelasi rezultat il poti obtine apeland din cod (sau in *Immediate Window*) metoda:

`System.AppDomain.CurrentDomain.GetAssemblies()`

Cu Helios, numarul de dll-uri necesare rularii aplicatiei se reduce la jumatate (20 fata de 39). Ramane insa valabila aceeasi afirmatie ca si la paragraful anterior: pe masura ce aplicatia va creste in complexitate, nr. de dll-uri ce apartin strict de aplicatie va creste, facand ca raportul dintre cele doua servere sa se diminueze treptat.

## Helios si System.Web, impreuna ##

Punand un *break point* pe ultima linie din `Startup.cs` si urmarind apelurile de pe stiva te poti convinge ca in pipeline-ul Http nu mai apare nicio referinta la System.Web. Cele 2 biblioteci instalate (Loader.IIS si Host.IIS) fac aproape toata treaba:

![](https://dl.dropboxusercontent.com/u/43065769/blog/images/2014/helios-callstack.png)

In schimb, daca verifici librariile incarcate in memorie ai sa ramai poate surprins. Mult criticatul System.Web este tot acolo. De ce? Biblioteca in sine, ofera inca multe functii utile. Probabil se poate elimina complet, nu stiu. Partea buna este ca acum rolul acestui modul s-a schimbat. Din comportamentul intrusiv pe care l-a avut inainte (se "agata" de fiecare request/response http) acum a devenit o biblioteca "docila" care raspunde doar atunci cand este intrebata.

![](https://dl.dropboxusercontent.com/u/43065769/blog/images/2014/helios-modules.png)

## Riscuri ##

A trecut mai bine de jumatate de an de cand proiectul Helios a fost expus publicului dar nici acum nu exista o versiune alfa sau beta. Nici macar cartitudinea ca acest proiect nu va fi abandonat. Personal, am decis sa-l folosesc in proiecte personale, de o mai mica anvergura. Daca aplicatia iti merge pe Helios, sigur te vei putea intoarce oricand la System.Web, fara nicio bataie de cap. Desigur, reciproca (tot ce merge pe System.Web merge si pe Helios) nu o nicidecum adevarata.  
Asadar, consider ca **riscul este minim**.


## Limitari ##

- Windows 8/Server 2012 + .Net 4.5.1 + Full trust
- WebApi only (nu merge pe MVC/Razor fiindca, prin modul in care a fost proiectat, MVC depinde de System.Web, deci de Asp.Net). De WebForms nici nu mai vorbesc.

Surse:

- [http://channel9.msdn.com/Series/Building-Modern-Web-Apps/07](http://channel9.msdn.com/Series/Building-Modern-Web-Apps/07) - Hanselman, Levi Broderick
- [Re-examining ASP.NET and Helios Performance Tests](Re-examining ASP.NET and Helios Performance Tests) - Rick Strahl
- [http://blogs.msdn.com/b/webdev/archive/2014/02/18/supplemental-to-asp-net-project-helios.aspx](http://blogs.msdn.com/b/webdev/archive/2014/02/18/supplemental-to-asp-net-project-helios.aspx) - Levi Broderick