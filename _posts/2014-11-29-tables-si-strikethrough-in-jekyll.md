---
layout: post
title:  "Tables si Strikethrough in Jekyll"
date:   2014-11-29 00:00:01
comments: true
categories: DevOps
---

In momentul de fata, specificatiile markdown [nu sunt standardizate](http://en.wikipedia.org/wiki/Markdown#Standardization). `Tables` si `Strkethrough` sunt optiuni care nu fac parte din [sintaxa originala](http://daringfireball.net/projects/markdown/syntax), dar se regasesc in multe din dialectele aparute ulterior. [GFM](https://help.github.com/articles/github-flavored-markdown/) (Github Flavored Markdown) este un astfel de dialect, care extinde sintaxa traditionala adaugand, printre altele, si suportul pentru [tables](https://help.github.com/articles/github-flavored-markdown/#tables) si [strikethrough](https://help.github.com/articles/github-flavored-markdown/#strikethrough).  



## Tables ##

Sintaxa:

```
| Aliniat stanga | Central       | Aliniat dreapta  |
| -------------- |:-------------:| ----------------:|
| text1          | text2         | $1600            |
| text-stanga    | text-centrat  | $12              |
```

Rezultatul:

| Aliniat stanga | Central       | Aliniat dreapta  |
| -------------- |:-------------:| ----------------:|
| text1          | text2         | $1600            |
| text-stanga    | text-centrat  | $12              |

Observatii:

- spatiile din stanga si din dreapta pipe-urilor (`"|"`) sunt optionale
- numarul caracterelor `"-"` care delimiteaza header-ul de body este tot optional
- alinierea textului se face cu ajutorul caracterului `":"`

## Strikethrough ##

Sintaxa:

```
Acesta este un ~~strikethrough~~ in markdown.
```

Rezultatul:

Acesta este un ~~strikethrough~~ in markdown.

## Configurari generale ##

Daca folosesti Redcarpet ca si motor de randare atunci trebui sa stii ca cele 2 facilitati (`tables` si `strikethrough`) nu sunt disponibile implicit. Ele trebuie [declarate](http://stackoverflow.com/a/16126840/2726725), sub forma unor extensii, in fisierul `_config.yml`:

```
markdown: redcarpet 
redcarpet:   
  extensions: ["tables", "strikethrough"]
```

In cazul `tables`, randarea (Redcarpet) transforma un text intr-un simplu `HTML table`, fara clase sau style-uri. Pentru formatarea tabelului eu am folosit urmatoarele style-uri:

```css
table {
    border-collapse: collapse;
    border-spacing: 0;
    margin-top:15px;
    margin-bottom:15px;
    border: 1px solid #ddd;
}

table th, td{
    padding: 6px 13px;
    border: 1px solid #ddd;
}

table th{
    background-color: #f8f8f8;
}
```

## Concluzie ##

`Tables` si `strikethrough` sunt optiuni utile dar mai rar folosite. Daca sintaxa celor doua comenzi le gasesti usor pe net, setarile din `_config.yml` sunt mai putin evidente. 
Sper ca informatia sa fie de folos si altora!

