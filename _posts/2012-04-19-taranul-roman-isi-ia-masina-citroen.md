---
layout: post
title:  "Taranul roman isi ia masina Citroen"
date:   2012-04-19 00:00:01
comments: true
categories: C#
---

Da, e tot despre programare!

Omul si marca din titlu sunt pur intamplatoare. Sau, ma rog, le-am ales, ca si celelalte cuvinte, doar pt. diacritice si semne de punctuatie.

Varianta slug a acestui titlu ar fi: "taranul-roman-isi-ia-masina-citroen" (vezi si URL-ul acestui articol). Codul C# care genereaza o astfel de transformare am ales sa-l prezint, alaturi de un live demo, folosind [recent anuntatul](http://blog.appharbor.com/2012/04/16/compile-net-in-your-browser-using-compilify-by-justin-rusbatch)  compilator online [Compilify.net](https://compilify.net/).

[Cod + Live Demo](https://compilify.net/pm/44) (comanda "run")

## Codul folosit: ##

```csharp
// ----------------------------------------------------------------------------------
// lucianmaran@hotmail.com, 19.04.2012, http://maran.ro/2012/04/19/taranul-roman-isi-ia-masina-citroen/
// ----------------------------------------------------------------------------------
// Description:
// 		Generate Slug (including accent removing) for a given phrase
// ----------------------------------------------------------------------------------
// Credits:
// 		Microsoft Tailspin Project (Codeplex)
// Other variants:
// 		Rob Conery, http://stackoverflow.com/questions/2095957/characters-to-strip-out-in-a-seo-clean-uri
// 		Azrafe7, http://stackoverflow.com/questions/249087/how-do-i-remove-diacritics-accents-from-a-string-in-net
// ----------------------------------------------------------------------------------

private static string GenerateSlug(string txt)
{
    string str = RemoveAccent(txt).ToLower();
    
    str = Regex.Replace(str, @"_", "-"); //otherwise two words separated by '_' become a single word
    
    str = Regex.Replace(str, @"[^a-z0-9\s-]", string.Empty);
    str = Regex.Replace(str, @"\s+", " ").Trim();
    
    str = str.Substring(0, str.Length <= 100 ? str.Length : 100).Trim();
    str = Regex.Replace(str, @"\s", "-");
    
    str = Regex.Replace(str, @"-+", "-").Trim(); //convert multiple '_' to single '_'
    
    return str;
}

private static string RemoveAccent(string txt)
{
    byte[] bytes = System.Text.Encoding.GetEncoding("Cyrillic").GetBytes(txt); //Tailspin uses Cyrillic (ISO-8859-5); others use Hebraw (ISO-8859-8)
    return System.Text.Encoding.ASCII.GetString(bytes);
}
```

## Test: ##

```csharp
return GenerateSlug("Ţăranul- român_îşi ia maşină... Citroën!");
```
## Rezultat: ##

```
"taranul-roman-isi-ia-masina-citroen"
```