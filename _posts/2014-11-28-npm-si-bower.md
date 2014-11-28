---
layout: post
title:  "NPM si Bower"
date:   2014-11-28 00:00:01
comments: true
categories: DevOps
---

[NPM](https://www.npmjs.org/) - este un ~~package~~ manager pt. librarii **server-side** (module Node.js)
[Bower](http://bower.io/) - este un package manager pt. librarii **client-side** (js, css). La urma urmei, tot un pachet npm.

Am remarcat, intamplator, ca exista o asemanare intre sintaxa comenzilor pt. cele doua tool-uri. Am vrut sa vad cat de departe merg aceste asemanari si asa am ajuns la urmatorul tabel comparativ:

### Manage this PM ###

|Command|NPM|Bower|Notes|
|---|:---:|---:|---|
|Install NPM/Bower|npm install npm -g|npm install bower -g|npm comes also with Node.js|
|Get version|npm -v|bower -v|-|
|Update NPM/Bower|npm update npm -g|npm update bower -g|for Windows, reinstalling Node.js might be a better option|

### Initialization ###

|Command|NPM|Bower|Notes|
|---|:---:|---:|---|
|Create config file|npm init|bower init|create and init. package.json / bower.json|



## Concluzie ##

In loc sa reinventeze propriile comenzi/optini, inginerii de la Twitter au ales sa refoloseasca, cat mai mult posibil, sintaxa din NPM. Iar acest lucru nu poate fi decat un avantaj pentru noi, programatorii.

