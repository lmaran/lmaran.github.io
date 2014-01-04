---
layout: post
title:  "Asta nu am stiut-o: Hoisting in JavaScript"
date:   2012-10-20 00:00:01
comments: true
categories: Javascript
---

Ce afiseaza codul de mai jos?

```javascript
var x = 5; //global
function func() {
    if (x == 5) {
        var x = 10;
        x++;
    }
}
func();
alert(x);
```

Nu, nu afiseaza "11"!

Afiseaza "5". Si asta pt. ca, in interiorul functiei, valoarea lui "x" in testul "if" este "undefined". Prin urmare, codul din interiorul blocului "if" nici nu se mai executa.
Dar de ce "x" este "undefined" cand, anterior, il defines pe "x" ca fiind o variabila **globala**?
Aici intervine notiunea de "**hoisting**" din titlu (hoist = ridicare).

In **C#**, domeniul de vizibilitate al unei variabile (scope) coincide cu **blocul** in interiorul caruia a fost definita, prin bloc intelegand orice secventa de cod cuprinsa intre accoladele "{" si "}".

In **JavaScript**, domeniul de vizibilitate al unei variabile coincide cu **functia** in interiorul careia a fost definita.
In plus, inainte de a fi executat, codul JavaScript este parsat si "rearanjat" a.i. toate declaratiile de variabile (nu si operatiile de atribuire) sunt mutate (ridicate) la inceputul zonei de vizibilitate (adica la inceputul functiei).

Acestea fiind spuse, codul initial este echivalenta cu:

```javascript
var x = 5; //global
function func() {
    var x; //declararea variabilei este "ridicata" la inceputul functiei (nu si initializarea). Variabila globala este suprascrisa.
    if (x == 5) { //variabila x inca nu este initializata (undefined) -> acest bloc nu se executa
        x = 10;
        x++;
    }
}
func();
alert(x);
```

**Concluzie:** Prin exempluul de mai sus am evidentiat conceptual de "**hoisting**" si am explicat modul in care difera aria de vizibilitate a unei variabile in JavaScript (**functie**) fata de C# (**bloc**).

**Recomandare:** In JavaScript, defineste toate variabile din interiorul unei functii la inceputul functiei. Initializarea acestor variabile o poti face oricand.

**Referinte:** [JavaScript, The Definitive Guide, 6th edition](http://www.amazon.com/JavaScript-Definitive-Guide-Activate-Guides/dp/0596805527), David Flanagan,  p. 53-55