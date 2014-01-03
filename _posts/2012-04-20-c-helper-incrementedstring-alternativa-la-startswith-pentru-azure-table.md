---
layout: post
title:  "C# Helper: IncrementedString() - alternativa la StartsWith() pentru Azure Table"
date:   2012-04-20 00:00:01
comments: true
categories: C# AzureTable
---

[Cod + Live Demo](https://compilify.net/r7/5) (comanda "run")

## Codul folosit: ##

```csharp
// ----------------------------------------------------------------------------------
// lucianmaran@hotmail.com, 20.04.2012, c-helper-incrementedstring-alternativa-la-startswith-pentru-azure-table
// ----------------------------------------------------------------------------------
// Description:
// 		Increment a given string (ex: "x2a" -> "x2b")
//		Usefull for "Start with" sorting in Azure Table Storage,http://www.dotnetsolutions.co.uk/blog/starts-with-query-pattern---windows-azure-table-design-patterns
// ----------------------------------------------------------------------------------
 
public static string IncrementedString(string str)
{
    if(String.IsNullOrEmpty(str)) 
        return" "; //first vizibile ch.
    
    char lastChar = str.Last();
    if(lastChar=='~') //last vizibil ch.
        return String.Format("{0}{1}",str.Substring(0,str.Length-1), "~ "); //add first visible ch.
    
    char incrementedLastChar = Convert.ToChar((Convert.ToInt32(lastChar) + 1));
    return String.Format("{0}{1}",str.Substring(0,str.Length-1), incrementedLastChar.ToString());
}
```

## Test: ##

```csharp
return IncrementedString("X2a");
```

Incrementeaza valoare unui string (ex: "x2a" -> "x2b").

Windows Azure Table nu suporta  interogari cu metoda StartsWith(). Prin urmare, singura solutie de a identifica acele inregistrari care incep cu un anumit substring presupune doua operatii de comparare:

- property/key >= substring
- property/key < substring “+1″ (evident, nu o simpla operatie aritmetica ci o operatie pe [coduri ASCII](http://www.asciitable.com/))
Problema este descrisa pe larg [aici](http://www.dotnetsolutions.co.uk/blog/starts-with-query-pattern---windows-azure-table-design-patterns) sau [aici](http://blogs.southworks.net/fboerr/2010/04/22/compsition-in-windows-azure-table-storage-choosing-the-row-key-and-simulating-startswith/), doar ca in fiecare din aceste cazuri sunt prezentate solutii particulare.

## Mod de utilizare: ##

![](https://dl.dropboxusercontent.com/u/43065769/blog/images/2012/IncrementedString.png)

## Setul de date: ##

![](https://dl.dropboxusercontent.com/u/43065769/blog/images/2012/DataDemo.png)

## Rezultatul: ##

![](https://dl.dropboxusercontent.com/u/43065769/blog/images/2012/JsonResult.png)