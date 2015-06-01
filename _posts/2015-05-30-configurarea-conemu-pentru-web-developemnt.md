---
layout: post
title:  "Configurarea ConEmu pentru Web Developemnt"
date:   2015-05-30 00:00:01
comments: true
categories: DevOps
---

Daca folosesti un simplu Code Editor (Sublime, Brackets, Atom, VSCode etc) in loc de un full IDE (VS, WebStorm etc), atunci consola joaca un rol imprtant in procesul de development.
Pe Windows, [ConEmu](http://conemu.github.io/) este o alternativa la clasicul `cmd`.

## Cerinte

Am sa prezint cum mi-am configurat eu aceasta noua consola. Cerintele au fost urmatoarele:

1. sa pot lansa 2 sesiuni separat, in acelasi ecran (split window): 
 - o sesiune in care sa rulez build-ul si server-ul de development (ex: `gulp dev`)
 - o sesiune de lucru (ex: pt. comenzi npm, gulp, bower etc)
2. ambele sesiuni sa ruleze implicit din directorul radacina al proiectului
3. sa pot configura cerintele de mai sus pt. fiecare proiect in parte (mai multe profile)
4. shortcut in taskbar pt. fiecare profil
5. sa am si o consola generica/default care sa porneasca automat intr-un folder prestabilit.

Rezultatul final, tradus in doua imagini, ar fi acesta:

![](https://dl.dropboxusercontent.com/u/43065769/blog/images/2015/05-30-conemu-shortut.png)

![](https://dl.dropboxusercontent.com/u/43065769/blog/images/2015/05-30-conemu-split-sessions.png)

## Configurarea profilelor

### 1. Configurarea unui profil 'default'

In `ConEmu`, fiecarui profil ii corespunde un `Task`. Toate proiectele mele sunt intr-un folder specific (ex: `C:\dadi\proiecte`). Prin urmare, as dori ca atunci cand lucrez pe un proiect care nu are definit un profil, pornirea consolei sa se faca automat in folder-ul `\proiecte`.

![](https://dl.dropboxusercontent.com/u/43065769/blog/images/2015/05-30-default-task.png)

```
	"cmd.exe" /k "%ConEmuBaseDir%\CmdInit.cmd" -new_console:d:C:\Data\Proiecte
```

Acest task il voi declara ca si `default` in meniul `Startup -> Specified named task`.

### 1. Configurarea unui profil specific unui anumit proiect

In acest caz, task-ul va avea 2 comenzi, cate una pt. fiecare sesiune:

![](https://dl.dropboxusercontent.com/u/43065769/blog/images/2015/05-30-project-task.png)

```
	> "cmd.exe" /k ""%ConEmuBaseDir%\CmdInit.cmd" &echo Starting gulp... & gulp" -new_console:d:C:\Data\Proiecte\Experimente\node-gulp
	
	"cmd.exe" /k "%ConEmuBaseDir%\CmdInit.cmd" -new_console:s50V:d:C:\Data\Proiecte\Experimente\node-gulp
	
```

Cateva observatii:

- caracterul `>` stabileste care va fi sesiunea activa (focus)
- odata lansata prima sesiune, se executa, succesiv, si urmatoarele comenzi:
	- `& echo` - afiseaza pe ecran un mesaj de start
	- `& gulp` - ruleaza comanda care porneste procesul de build (si server-ul de development)
- se observa totodata si 2 setari specifice consolei:
	- `s50V` - cele 2 sesiuni vor apare pe acelasi ecran, cu split vertical la 50%
	- `d:<path>` - stabileste directorul default 
	
## Adaugarea unui shortcut in taskbar

Un shortcut corespunde unui anumit `task`. Adauga task-ul dorit la o lista de "history commands", astfel: 

![](https://dl.dropboxusercontent.com/u/43065769/blog/images/2015/05-30-new-console-dialog.png)

Task-urile din acest "history list" devin accesibile doar dupa pornirea consolei (vezi poza anterioara).
Daca insa doresti sa vizualizezi aceste task-uri si sub forma de shortcut-uri in taskbar, atunci trebuie sa configurezi:

![](https://dl.dropboxusercontent.com/u/43065769/blog/images/2015/05-30-add-history-to-taskbar.png)

In acest moment poti apela consola, pentru fiecare proiect in parte, direct din taskbar (vezi prima poza).

## Concluzie

Pe langa window resize, tabbed window, copy/paste, color schemes etc, [ConEmu](http://conemu.github.io/) iti permite si un acces rapid (shortcut-uri) la mai multe instante personalizate.

**Bonus**: Daca te intrebi ce tema (schema de culori) am folosit: `Settings -> Features -> Colors -> Monokai`