---
layout: post
title: "MyBlog - customizare cu ThemeFrame"
date: 2010-11-21 00:06:31
comments: true
---

Daca doresti pt. WordPress o tema customizabila atunci nu poti ignora [Atahualpa](http://wordpress.org/themes/atahualpa "Atahualpa"), cu aproape 300 de optiuni si 650.000 de download-uri. Daca doresti mai mult de atat, autorii acestei populare teme [bytesforall](http://wordpress.bytesforall.com) au realizat o aplicatie comerciala [ThemeFrame](http://themeframe.com) - (80$) care iti permite, folosind cunostinte minime de CSS, HTML, JavaScript si PHP, sa-ti construiesti propria tema.

## Instalare

Instructiunile de instalare se gasesc [aici](http://forum.bytesforall.com/showthread.php?t=9143). Cu serviciile din WAMP pornite pe PC-ul local (Apache, SQLLite, PHP), aplicatia se acceseaza din browser: _http://localhost/themeframe_ (in varianta Beta am lucrat cu Mozilla; cu IE nu a functionat). Locatia fizica implicita unde sunt localizate fisierele aplicatiei este "_...\wamp\www\themeframe_".

## Modul de functionare al aplicatiei

Aplicatia incepe cu o customizare "basic" ale carei setari sunt preluate dintr-o BD SQLLite (fisierul "_...\wamp\www\themeframe\sqlite\thfrdb_"). La orice operatie de salvare, modificarile efectuate se memoreaza in aceeasi BD SQLLite. Fisierele aferente noii teme se creaza "on-the-fly", in momentul in care se face publicarea prin FTP. Exista si varianta in care aceste fisiere pot fi generate manual, din aceeasi sursa, folosind meniurile "_Get Whole Theme as ZIP_" sau "_Get Single Theme Files_".

Atentie! ThemeFrame nu poate edita o tema plecand de la fisierele acesteia (php, css, js, imagini etc). Nici macar atunci cand aceste fisiere au fost generate cu ThemeFrame. Exista doar 2 metode de a importa o tema:

- varianta oficiala: se importa fisierul "_thfr-settings-localhost\*.txt_" exportat in prealabil din aceeasi aplicatie
- varianta neoficiala: se suprascrie fisierul aferent bazei de date cu o varianta anterior salvata (fisierul "_...\wamp\www\themeframe\sqlite\thfrdb_")

La transferul prin FTP se copiaza doar fisierele de configurare, nu si fisierele media (ex: poze). Acestea tb. copiate manual in folderul blog-ului ("_.../wp-content/themes/my-new-theme/images/_").

Atentie! Modificari ale temei NU se vor face direct din sectiunea "_wp-admin_" a site-ului decat pt. testare. In caz contrar, modificarile astfel facute vor fi suprascrise la primul transfer prin FTP facut din ThemeFrame.

## Setari pt. FTP-Transfer

Setarile pt. server, user, parola si nume-tema sunt clare. Pentru "_Path to WordPress Themes directory_" am introdus calea "_maran.ro/wwwroot/blog/wp-content/themes_". Asta fiiindca blog-ul nu l-am instalat direct in radacina (wwwroot) ci intr-un folder situat pe nivelul imediat superior (_blog_).

## Modificari personale aduse la framework

1. **Memorare parola pt. FTP-Transfer**. Implicit, la salvarea setarilor din sectiunea "FTP-Transfer" se memoreaza toate valorile, mai putin parola de la contul de FTP. Ca sa nu fiu nevoit sa introduc parola la fiecare update, am facut urmatoarea modificare in fisierul tf_ftp_transfer.php:

- `$user_pass = $_POST['ftp_user_pass'];` (old)
- `$user_pass = 'parola';` (new)

2. **Localizar in RO**: in principiu, toate denumirele se pot modifica prin editarea fisierului "_includes/tf_create_php_file.php_".

## Rezultatul final

![my blog theme](/assets/images/2010/Myblog-theme.png)
