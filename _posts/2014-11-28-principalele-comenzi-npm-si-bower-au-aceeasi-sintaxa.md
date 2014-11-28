---
layout: post
title:  "Principalele comenzi NPM si Bower au aceeasi sintaxa"
date:   2014-11-28 00:00:01
comments: true
categories: DevOps
---

[NPM](https://www.npmjs.org/) - este un package manager pt. librarii **server-side** (module Node.js)

[Bower](http://bower.io/) - este un package manager pt. librarii **client-side** (js, css). La urma urmei, tot un modul npm.

Am remarcat, intamplator, ca exista o asemanare intre sintaxa comenzilor pt. cele doua tool-uri. Am vrut sa vad cat de departe merg aceste asemanari si asa am ajuns sa adun datele de mai jos.

**Atentie!** - am amintit doar comenzile si optiunele pe care le-am considerat mai importante.

### Install tool ###

| Action | NPM | Bower | Obs |
| -------| --- | ----- | --- |
| Install NPM/Bower | `npm install npm -g` | `npm install bower -g` | npm comes also with Node.js |
| Get version | `npm -v `| `bower -v` | |
| Update NPM/Bower | `npm update npm -g` | `npm update bower -g` | for Windows, reinstalling Node.js might be a better option |

### Initialization ###

| Action | NPM | Bower | Obs |
| ------ | --- | ----- | --- |
| Create config file | `npm init` | `bower init` | create and init. package.json / bower.json |

### Install package ###

| Action | NPM | Bower | Obs |
| ------ | --- | ----- | --- |
| Install local | `npm install [package]` | `bower install [package]` | |
| Install global | `npm install [package] -g` | N/A | |
| Install local, all packages | `npm install` | `bower install` | based on package.json / bower.json |
| Install global, all packages | `npm install -g` | N/A | based on package.json / bower.json |

### Install options ###

| Action | NPM | Bower | Obs |
| ------ | --- | ----- | --- |
| Save pck reference in `dependencies` section | `--save` | `--save` | |
| Save only in `devDependencies` section | `--save-dev` | `--save-dev` | |
| Do not install pck from `devDependencie` section | N/A | `--production` | |
| Fetch remote resources even if a local copy exists | `--force` | N/A | |

### Update package ###

| Action | NPM | Bower | Obs |
| ------ | --- | ----- | --- |
| Update local | `npm update [package]` | `bower update [package]` | |
| Update global | `npm update [package] -g` | N/A | |
| Update local, all packages | `npm update` | `bower update` | based on package.json / bower.json |
| Update global, all packages | `npm update -g` | N/A | based on package.json / bower.json |


### List installed packages (and their actual and latest version) ###

| Action | NPM | Bower | Obs |
| ------ | --- | ----- | --- |
| List all local packages | `npm list` | `bower list` | or [npm list --depth=0](http://stackoverflow.com/a/16704412/2726725) (npm only) |
| List all global packages | `npm list -g` | N/A | or [npm list -g --depth=0](http://stackoverflow.com/a/16704412/2726725) (npm only) |
| Get info for a specific local package | `npm list [package]` | N/A | |
| Get info for a specific global package | `npm list [package] -g` | N/A | |
| Get outdated local packages | `npm outdated` | N/A | |
| Get outdated global packages | `npm outdated -g` | N/A | |

### Cache management ###

| Action | NPM | Bower | Obs |
| ------ | --- | ----- | --- |
| Display all packages in cache | `npm cache list` | `bower cache list` | |
| Clear all packages from cache | `npm cache clean` | `bower cache clean` | |
| Display package in cache | `npm cache list [path]` | `bower cache list` | |
| Clear package from cache | `npm cache clean [path]` | `bower cache clean [path]` | |


### Prune (remove "extraneous" packages) ###

Extraneous packages - sunt pachetele care nu mai au referinte in fisierul package.json / bower.json dar exista inca pe disk

| Action | NPM | Bower | Obs |
| ------ | --- | ----- | --- |
| Delete all extraneous packages | `npm prune` | `bower prune` | |
| Delete specific extraneous package | `npm prune [package]` | N/A | |

### Search ###

| Action | NPM | Bower | Obs |
| ------ | --- | ----- | --- |
| Search in remote repository | `npm search <term>` | `bower search <term>` | |

### Help ###

| Action | NPM | Bower | Obs |
| ------ | --- | ----- | --- |
| General help | `npm help` | `bower help` | |
| Help for specific commands| `npm help [command]` | `bower help [command]` | |

Si fiindca tot am vorbit mai sus de pachetele npm globale, aceste le poti vedea in folder-ul `C:\Users\Lucian\AppData\Roaming\npm.`. Calea spre acest folder apare si la rularea comenzii `npm list -g --depth=0`.



## Concluzie ##

In loc sa reinventeze propriile comenzi/optini, inginerii de la Twitter au ales sa refoloseasca, cat mai mult posibil, sintaxa din NPM. Iar acest lucru nu poate fi decat un avantaj pentru noi, programatorii.

