---
layout: post
title: "CRM2011 - Acces de 10 ori mai rapid cu MicroORM (PetaPoco) fata de XRM SDK"
date: 2012-07-15 00:00:01
comments: true
categories: C# MS-CRM PetaPoco
---

Da, de 10 ori! Sau chiar mai mult.

[Ma asteptam](http://www.toptensoftware.com/Articles/94/PetaPoco-More-Speed) ca accesul direct la SQL (cu ADO.NET sau orice wrapper peste acesta (Dapper, PetaPoco, Massive etc)) sa fie net superior serviciilor oferite de platforma XRM atunci cand vine vorba de performanta bruta. Dar nu ma asteptam ca aceasta diferenta sa fie atat de evidenta.

Testele le-am facut doar la citire, fiindca doar aceasta operatie ma intereseaza (a scrie direct in baza de date SQL este posibil, dar total nerecomandat). Mai precis, am incercat sa returnez o inregistrare cu cateva campuri de pe entitatea Invoice. Dar hai sa vedem intai rezultatele:

- XRM-SDK: ~125 ms

![](/assets/images/2012/InvoiceInBrowser-XRM-SDK.png)

- PetaPoco: ~15ms

![](/assets/images/2012/InvoiceInBrowser-PetaPoco.png)

De unde provine diferenta? Sa fie codul SQL generat de serviciile XRM mai putin eficient decat in cazul PetaPoco? Nu, ambele variante au practic aceeasi "amprenta" (testat cu SQLProfiler):

- XRM-SDK:

![](/assets/images/2012/InvoiceInSQL-XRM-SDK.png)

- PetaPoco:

![](/assets/images/2012/InvoiceInSQL-PetaPoco.png)

Se observa ca ambele coduri returneaza aceleasi campuri, din aceeasi tabela, cu acelasi parametru. Difera doar "estetica".

Hai sa mergem putin mai departe si sa analizam traficul generat in cele 2 situatii. Pt. asta am folosit Wireshark, instalat pe server-ul de API. In felul acesta pot vedea atat cererile (ma intereseaza doar cele HTTP) pe care server-ul le primeste din partea clientilor cat si schimbul de mesaje pe care server-ul il face cu alte servicii (in cazul de fata serviciul XRM). Sa facem, din nou, o radiografie a celor 2 situatii:

- Trafic – XRM SDK:

![](/assets/images/2012/InvoiceInHTTP-XRM-SDK.png)

- Trafic – PetaPoco:

![](/assets/images/2012/InvoiceInHTTP-PetaPoco.png)

La prima vedere, pozele de mai sus pot parea putin cam criptice, dar sa le luam pe rand:

In cazul XRM, prima inregistrare (nr.133) reprezinta cererea sosita din partea clientului (setat ca si moment de referinta), iar ultima inregistrare (nr.490) reprezinta raspunsul server-ului catre client (dupa 137 ms, in acest caz). Pentru a deservi cererea, server-ul, la randul lui, apeleaza la serviciul XRM. Acest lucru se face prin 4 cereri HTTP (2 de tip GET si doua de tip POST), trafic evidentiat cu albastru deschis. Durata necesara prelucrarii acestor cereri este inregistrata in coloana 2 (exprimata in secunde scurse de la momentul de referinta).

In cazul **PetaPoco**, server-ul API acceseaza direct baza de date CRMsql. Se vede ca raspunsul e generat in 6 ms. Pentru a vedea timpul real in care raspunsul ajunge la client, la cele ~6ms (sau 13 ms pt. primul caz) se adauga si latenta retelei, adica durata in care pachetele parcurg distanta client-server si retur.

## Concluzie

Ca serviciul XRM ofera un timp de raspuns nu foarte grozav, este evident. Daca sunteti familiarizati cu SDK-ul platformei XRM si performanta pt. modulele pe care le scrieti nu este critica, folositi-l! Acesta este drumul “batatorit”. Sa retinem totusi ca avem si o alternativa: accesul direct la baza de date (SQL) a CRM-ului. In acest caz performanta este mult superioara, doar ca scenariile in care aceasta ultima metoda poate fi aplicata trebuie bine analizate: nu o vom folosi, de exemplu, pt. operatii de scrire, dar nici pt. operatii de tip citire a unor inregistrari care necesita autorizare la nivel de utilizator.

## Bonus

Codul scris de mine pt. testarea in cele 2 scenarii a fost de **2 ori mai compact, cu PetaPoco** fata de cazul clasic cu XRM SDK. Practic 2 linii (20 si 26). De restul se ocupa microORM-ul amintit:

- managementul conexiunii la baza de date
- selecturea automata a tabelei si campurilor necesare pt. interogare
- depozitare automata a rezultatelor intr-un obiect c#

```csharp
using System; //PetaPoca.cs adaugat ca fisier separat in proiect (fara dll)

namespace AppStudio.Infrastructure.Data.MainBoudedContext.CRMmodule.Repositories
{

    [PetaPoco.TableName("Invoice")] //util doar daca nume clasa != nume tabela SQL
    public class CrmInvoice2
    {
        public Guid InvoiceId { get; set; }

        [PetaPoco.Column("new_SalesPersonId")] //util doar daca nu proprietate != nume coloana SQL
        public Guid SalesPersonId { get; set; }

        [PetaPoco.Column("new_SalesPersonIdName")]
        public string SalesPersonName { get; set; }
    }

    public class CrmInvoiceRepository2
    {
        private PetaPoco.Database db = new PetaPoco.Database("CRMConnectionString"); //din web.config

        public CrmInvoice2 GetCrmInvoice2(Guid id)
        {
            try
            {
                return db.SingleOrDefault<CrmInvoice2>("WHERE InvoiceId= @0", id); //select automat; bind automat
            }
            catch (Exception ex)
            {
                throw ex;
            }
        }
    }

}
```
