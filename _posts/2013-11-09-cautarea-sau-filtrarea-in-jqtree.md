---
layout: post
title: "Cautarea (sau filtrarea) in jqTree"
date: 2013-11-09 00:00:01
comments: true
categories: javascript
---

In ultimul proiect am folosit biblioteca [jqTree](http://mbraak.github.io/jqTree/) pt. a afisa intr-o pagina web un set de date organizate arborescent.

M-a interest sa pot face cautari (filtrari) in arborele generat, dar cum jqTree nu ofera aceasta facilitate (si nici altundeva pe net nu am gasit) am hotarat sa o implementez singur.

[Online demo](http://jsfiddle.net/lmaran/XL4S6/15/)

![](/assets/images/2013/jqTreeSearchResult.png)

## Facilitati expuse:

- compararea se face doar pe nodurile terminale ('frunze')
- case insensitive
- cautare partiala (in interiorul cuvantului)
- numar nelimitat de niveluri

## Detalii de integrare:

JqTree expune proprietatea [onCreateLi](http://mbraak.github.io/jqTree/#tree-options-oncreateli) prin care iti permite sa atasezi o functie ‘custom’ care sa poata fi executata inainte de randarea fiecarui nod. Tot ceea ce trebuie sa faci este sa copiezi codul de mai jos si sa-l alaturi la restul optiunilor folosite la initializarea tree-ului (vezi si codul sursa de la [demo](http://jsfiddle.net/lmaran/XL4S6/15/)).

```javascript
onCreateLi: function (node, $li) {
    var searchValue = $('#searchField').val().toLowerCase();
    var nodeValue = $li.find('.jqtree-title').text().toLowerCase();

    if ($li.find('.jqtree-toggler').length == 0) { // child (leaf)
        if (nodeValue.indexOf(searchValue) == -1) { //child mismatch
            $li.find('.jqtree-title').hide(); //hide child
        }
        else { //child match
            //show child and all parent folders, down to root
            for (var i = node.parent.getLevel() ; i > 0; i--) {
                $(node.parent.element).find("div").first().show();
                node.parent = node.parent.parent;
            }
        }
    }
    else { // folder
        $li.find('.jqtree-element').hide(); //initial hide all folders
    }
}
```

Cautarea in tree (refresh-ul) se face cu comanda [loadData](http://mbraak.github.io/jqTree/#functions-loadData) iar declansarea acestei comenzi se poate face in mai multe feluri ('_on button Click_', '_on Enter key_' sau '_as you type_'). Pentru ultimul scenariu, codul este urmatorul:

```javascript
// search on type
// http://stackoverflow.com/questions/10318575/jquery-search-as-you-type-with-ajax
var thread = null;
$("#searchField").keyup(function () {
  clearTimeout(thread);
  var searchVal = $(this).val();
  thread = setTimeout(function () {
    $("#tree1").tree("loadData", data);
  }, 100);
});
```

Succes!

**Update (23.11.2013)**: In exemplul de mai sus am considerat ca doar 'frunzele' arborelului sunt purtatoare de informatie utila, nodurile de tip 'folder' avand doar rol de legatura. Daca te intereseaza ca filtrarea sa se faca pe **toate** nodurile, recomand solutia prezentata de FKobus [aici](https://github.com/mbraak/jqTree/issues/230).
