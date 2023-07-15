---
layout: post
title: "CRM 4.0 - eroare la rapoartele modificate cu BI-ul din SQL2008 R2"
date: 2010-12-14 00:00:01
comments: true
---

## Evidentierea problemei

- Export orice raport din MS CRM 4.0
- Deschid raportul (fisierul .rdl) cu Visual Studio – componente de Business Intelligence (BI) din SQL2008 R2
- Fac orice modificare si salvez
- Import cu succes – raportul din nou in CRM
- Rulez raportul si primesc urmatoarea eroare (event 19969 din App. Log):

![](/assets/images/2010/ErrRaportt.png)

- Aceeasi eroare primesc si daca creez un raport nou in care folosec param. standard CRM (Ex: CRM-URL)

## Cauza

Parametrii folositi pt. transmiterea de informatii de la CRM catre raport sunt pastrati/setati – cum e si normal – pe hidden dar in momentul in care raportul este (re)importat in CRM aceasta valoare nu se pastreaza (posibil un bug).

Exemplu: param. CRM_URL folosit in raport pt. drilldown are, inainte de import atributul `hidden=true` (din Visual Studio sau direct din sursa fisierului):

![](/assets/images/2010/HiddenTrue1.png)

![](/assets/mages/2010/HiddenTrue2.png)

Dupa import, raportul importat in CRM va avea atributul `hidden=false` (din http://.../reports)

## Solutia

Bifezi optiunea `Hide` din figura de mai sus. Mai multe detalii [aici](http://blog.customereffective.com/blog/2010/08/crm-pre-filtered-reports-and-ssrs-p1-parameter-error.html)!
