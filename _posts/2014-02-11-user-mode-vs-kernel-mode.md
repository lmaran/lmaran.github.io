---
layout: post
title:  "User Mode vs. Kernel Mode"
date:   2014-02-11 00:00:01
comments: true
categories: Infrastructure, IIS
---

Pe aproape orice diagarama care descrie **arhitectura IIS** apar cele doua notiuni din titlu. Spre axemplu: [aici](http://www.iis.net/learn/get-started/introduction-to-iis/introduction-to-iis-architecture), [aici](http://blogs.msdn.com/b/friis/archive/2012/08/13/iis-7-5-architecture-and-components-part-1.aspx) sau [aici](http://www.codeproject.com/Articles/28693/Deploying-ASP-NET-Websites-on-IIS-7-0). Fiindca a trecut ceva vreme de cand am terminat scoala, am decis sa-mi clarific, din nou, aceste notiuni.

Atunci cand vorbim despre `kernel mode` si `user mode` trebuie sa stabilim mai intai contextul. Aceasta terminologie apare atat la nivel de hardware (CPU) cat si la nivel de software (SO, aplicatii), in stransa legatura una cu cealalta. 

## 1. User/Kernel mode - la nivel de CPU ##

In principal procesoarele (ma refer aici la Intel) folosesc 2 moduri de operare:

### Kernel mode (Ring 0) ###

 - numit si *protected mode*, *supervizor mode* sau *priviledged mode*
  - ofera access full la toate resursele (set de instructiuni, porturi, zone de momorie etc)
 - este modul de care beneficiaza SO Windows si driverele care necesita acces direct la resurse (ex: drv. video acceseaza direct memoria video)
 - o eroare aparuta la nivel de kernel blocheaza intregul PC
 
### User mode (Ring 3) ###

 - numit si *real mode*
 - Ring 1 si 2 sunt in general nefolosite
 - ofera un acces limitat la resurse. Spre exemplu, majoritatea proceselor  si aplicatiilor instalate pe Windows implica comutarea CPU-ului in `user mode`, primind astfel acces doar la o anumita parte din memorie. Aceasta memorie se mai numeste si **memorie virtuala** pt. ca procesul este "pacalit" sa creada ca i s-a pus la dispozitie intreaga memorie reala. Mai mult, nici adresele pe care le foloseste aplicatia nu sunt de cele reale ci sunt decalate cu un "offset" fata de mem. fizica.
 - neavand zone comune, orice eroare aparuta la nivelul unui proces nu afecteaza celelalte procese

Alte observatii: 

- comutarea CPU-ului dintr-un mod in altul se realizeaza prin intermediul unor [intreruperi](http://en.wikipedia.org/wiki/Inter-processor_interrupt). 
- operatia de *comutare* implica un anumit cost (necesita o salvare a  contextului a.i., la revenire, CPU-ul sa poata continua operatiile din locul in care a ramas)

 ![](https://dl.dropboxusercontent.com/u/43065769/blog/images/2014/cpu-rings.png)

## 2. User/Kernel mode - la nivel de SO (Windows) / aplicatii ##

### Kernel mode ###

 - partea cea mai importanta din SO Windows ([HAL](http://en.wikipedia.org/wiki/Hardware_abstraction)-ul, kernel-ul, majoritatea driverelor etc) opereaza in `kernel mode`. Exceptie fac procesele care se instaleaza odata cu Windows-ul. Acestea operaza in `user mode`, ele nefiind  altceva decat niste utilitare obisnuite.  

  ![](https://dl.dropboxusercontent.com/u/43065769/blog/images/2014/windows-user-and-kernel-mode.png)

### User mode (Ring 3) ###

 - majoritate aplicatiile instalate de utilizator opereaza un `user mode`. Exceptie fac, printre altele:
  - driver-ul video
  - modulul HTTP.sys din IIS (pt. a asigura o mai buna performanta):

  ![](https://dl.dropboxusercontent.com/u/43065769/blog/images/2014/iis-architecture-user-kernel-mode.png) 

Sursele imaginilor si alte referinte:

- [http://www.codinghorror.com/blog/2008/01/understanding-user-and-kernel-mode.html](http://www.codinghorror.com/blog/2008/01/understanding-user-and-kernel-mode.html)
- http://msdn.microsoft.com/en-us/library/windows/hardware/ff554836(v=vs.85).aspx
- [http://en.wikibooks.org/wiki/Windows_Programming/User_Mode_vs_Kernel_Mode](http://en.wikibooks.org/wiki/Windows_Programming/User_Mode_vs_Kernel_Mode)
- [http://en.wikipedia.org/wiki/CPU_modes](http://en.wikipedia.org/wiki/CPU_modes)
- [http://blogs.msdn.com/b/friis/archive/2012/08/13/iis-7-5-architecture-and-components-part-1.aspx](http://blogs.msdn.com/b/friis/archive/2012/08/13/iis-7-5-architecture-and-components-part-1.aspx)