---
layout: post
title:  "Windows Server 2008R2: One-way Forest Trust"
date:   2011-11-09 00:00:01
comments: true
categories: IT-infrastructure
---

Am doua forest-uri, fiecare cu cate un singur domeniu:

- **Domeniul A** este domeniul principal al companiei in care se autentifica toti angajatii, izolat strict de mediul extern.
- **Domeniul B** este un domeniu care contine resurse (fisiere, baze de date) ce pot fi accesate de anumite servere interne (ex: un CRM2011 in configuratie IFD) care publica continut in Internet.
Pentru teste si deploy de software, grupul de dezvoltatori din  forest A trebuie sa aibe acces la masinile din forest B. In acelasi timp, nici un cont din B nu trebuie sa poata accesa resurse din A. Mai jos am reprezentat, schematic, o astfel de relatie one-way intre cele 2 forest-uri:

 ![](https://dl.dropboxusercontent.com/u/43065769/blog/images/2011/ForestTrust.png)

B are incredere in A, si astfel, utilizatorii din A pot accesa resurse in B.


**Solutia 1**: Fiecare dezvoltator din A are un cont suplimentar in  B. Dezavantaj: fiecare incercare de acces este intrerupta de o etapa de autentificare. Plus memorarea unor credentiale suplimentare.

**Solutia 2**: Stabilirea unei relatii de trust, unidirectionala, intre cele 2 forest-uri. In continuare am sa descriu, pe scurt, cum se realizeaza acest lucru:

 
## 1. Stabilirea relatiei de trust ##

### Configurari pe DomainController-ul (DC) din B: ###

- Adauga in server-ul de DNS o zona de tip "Stub zone" catre zona din A (se adauga astfel doar inregistrarile relevante, necesare pt. a putea contacta DC-ul din dom. A

 ![](https://dl.dropboxusercontent.com/u/43065769/blog/images/2011/image.png)

- Din “AD Domain and Trust” verifica ca “Raise Forest Function Level” si “Raise Domain Functional Level” sa fie setate pe nivelul Win.2008 R2 (recomandat).
- Din “AD Domain and Trust” adauga o relatie de tip  “trust” + “outgoing” catre domeniul de incredere (dom. A). Accepta sa se creeze automat setarile la “capatul” celalalt (adica in A). La final, valideaza conexiunea (butonul Validate)

 ![](https://dl.dropboxusercontent.com/u/43065769/blog/images/2011/image1.png)
 
### Configurari pe DC-ul din A: ###
- Daca la pasul anterior ai acceptat ca setarile din dom.A sa se creeze automat, poti sa mergi direct pe dom. A si ai sa regasesti asta:

 ![](https://dl.dropboxusercontent.com/u/43065769/blog/images/2011/image2.png)

- La final, valideaza conexiunea (butonul Validate)
 
## 2. Acordare drepturilor ##

### Configurari pe DC-ul din A: ###
- Creaza un Global Group (ex: TrustedGroup**For**A)
- Adauga in grupul de mai sus toti userii din A care urmeaza sa acceseze resurse din B

### Configurari pe DC-ul din B: ###

- Creaza un Domain Local Group (ex: TrustedGroup**From**A)
- Adauga in grupul de mai sus, grupul creat anterior in dom.A. Butonul Locations iti va permite sa faci "browse" si in acest domeniu:

 ![](https://dl.dropboxusercontent.com/u/43065769/blog/images/2011/image3.png)

- Atribuie permisiuni acestui grup la resursele dorite.

- Observatii generale legate de acordarea drepturilor:
 
 Nu poti sa adaugi useri/grupuri din afara forest-ului intr-un grup de dip Global. Prin urmare, nici in grupul "**DomainAdmin**" nu poti adauga membrii din alt forest. Singura varianta de a da drepturi maxime este sa adaugi grupurile/userii din alt forest in grupurile locale numite "Administrators". Fiind un grup local, acest lucru trebuie facut pe fiecare masina (sau daca ai mai multe masini poti automatiza procesul cu GPO). Desigur, aceste drepturi vor fi inferioare celor de tip DomainAdmin.
 

## 3. Stabilirea permisiunilor ##

- Daca in pasul anterior s-au stabilit drepturi de "Administrators" pe o anumita masina din dom.B, atunci poti intra "remote" (RDP) pe respectiva masina cu contul din dom.A.
- Pentru acordarea permisiunilor pe obiecte (ex: un share de folder) poti apel la grupul "TrustedGroup**From**A" din dom.B. Desi nu se recomanda, poti alege sa folosesti si direct grupul "TrustedGroup**For**A" din dom.A, cu dezavantajul ca trebuie sa reintroduci credentialele pt. dom.A.
 
## Pas 4. Accesul la resurse ##

Presupunem ca un utilizator de pe o masina din dom.A (ex: `pc.domeniulA.ro`) doreste sa acceseze o masina din dom.B (ex: `srv.domeniuB.ro`).

- configurezi srv. de DNS din dom.A ca cererile sosite pe someniulB.ro sa le trimita la srv. de DNS din dom.B (zona de tip "Conditional Forwarders")

 ![](https://dl.dropboxusercontent.com/u/43065769/blog/images/2011/image4.png)

- configurezi statiile din dom.A ca la toate interogarile sa adauge automat dns-sufixul "domeniulB.ro" (optional, pt. apel doar cu nume, fara sufix). Poti automatiza procesul cu GPO sau, poti face operatia manual, la nivel de statie:

 ![](https://dl.dropboxusercontent.com/u/43065769/blog/images/2011/image5.png)

Cu modificari usor de intuit, ghidul de mai sus poate fi folosit si pt. configurarea unei legaturi de tipul **two-way**.