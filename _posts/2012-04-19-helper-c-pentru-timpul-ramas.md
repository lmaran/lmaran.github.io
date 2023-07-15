---
layout: post
title: "C# Helper: RemainingTime() - pentru sortare implicita in Azure Table"
date: 2012-04-19 00:00:02
comments: true
categories: C# AzureTable
---

Am creat cateva metode simple care returneaza timpul scurs pana la o anumita data, in diverse unitati de masura: ticks, secunde, minute etc.

[Cod + Live Demo](https://compilify.net/qh/11) (comanda "run")

## Codul folosit:

```csharp
// ----------------------------------------------------------------------------------
// lucianmaran@hotmail.com, 19.04.2012, http://maran.ro/2012/04/19/helper-c-pentru-timpul-ramas/
// ----------------------------------------------------------------------------------
// Description:
// 		Generate remaining time until 31/12/2042 (ticks, seconds, minutes etc)
//		Usefull for default descendent sorting in Azure Blobs or Tables, http://blog.liamcavanagh.com/2011/11/how-to-sort-azure-table-store-results-chronologically/
// ----------------------------------------------------------------------------------

static readonly DateTime yearEnd = new DateTime(2042, 12, 31);
public static long Ticks()
{
    return yearEnd.Ticks - DateTime.Now.Ticks;
}
public static long Seconds()
{
    TimeSpan remainingSeconds = new TimeSpan(Ticks());
    return (long)remainingSeconds.TotalSeconds;
}
public static long Minutes()
{
    TimeSpan remainingSeconds = new TimeSpan(Ticks());
    return (long)remainingSeconds.TotalMinutes;
}
public static long Hours()
{
    TimeSpan remainingSeconds = new TimeSpan(Ticks());
    return (long)remainingSeconds.TotalHours;
}
public static long Days()
{
    TimeSpan remainingSeconds = new TimeSpan(Ticks());
    return (long)remainingSeconds.TotalDays;
}
```

## Test:

```csharp
StringBuilder sb = new StringBuilder();

sb.AppendLine("");
sb.AppendLine(String.Format("Remaining time until {0:dd/MM/yyyy}", yearEnd));
sb.AppendLine("----------------------------------");
sb.AppendLine(String.Format("{0,-10}{1,-20}({2})", "Ticks:", Ticks(), Ticks().ToString().Length));
sb.AppendLine(String.Format("{0,-10}{1,-20}({2})", "Seconds:", Seconds(), Seconds().ToString().Length));
sb.AppendLine(String.Format("{0,-10}{1,-20}({2})", "Minutes:", Minutes(), Minutes().ToString().Length));
sb.AppendLine(String.Format("{0,-10}{1,-20}({2})", "Hours:", Hours(), Hours().ToString().Length));
sb.AppendLine(String.Format("{0,-10}{1,-20}({2})", "Days:", Days(), Days().ToString().Length));

return sb;
```

Un exemplu de rezultat este cel de mai jos:

![](/assets/images/2012/RemainingTime.png)

## La ce sunt bune aceste cifre?

- ca prefix la PK/RK pentru sortare implicita in Azure Tables
- ca prefix la slug-ul unei resurse web (unicitate URL poza)
- atunci cand vrei sa generezi un ID mai “prietenos” decat un GUID
- etc

## De ce 2042?

Am considerat acest an ca fiind un bun compromis intre numarul de cifre necesare reprezentarii numerelor de mai sus si durata de viata maxima a unei aplicatii. Spre exemplu, daca "urc" la 2043, numarul de cifre creste cu o unitate pt. Ticks si Secunde. Daca vreau sa scad numarul de cifre cu o unitate, trebuie sa "cobor" anul la 2014 (sper ca aplicatia mea sa fie in functiune si dupa aceasta data -:)

[Puteti simula](https://compilify.net/qh/11) si alte variante, ruland codul cu diverse valori pentru anul final (yearEnd).
