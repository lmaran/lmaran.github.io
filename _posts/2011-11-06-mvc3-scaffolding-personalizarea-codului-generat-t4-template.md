---
layout: post
title: "MVC3 Scaffolding - personalizarea codului generat (T4 Template)"
date: 2011-11-06 00:00:01
comments: true
categories: MVC
---

Pt. generarea automata a codului corespunzator controller-ului si view-urilor, folosesc optiunea de _Scaffolding_ introdusa in MVC3 Tools Update. ScottGu descrie f. bine [aici](http://weblogs.asp.net/scottgu/archive/2011/05/05/ef-code-first-and-data-scaffolding-with-the-asp-net-mvc-3-tools-update.aspx) la ce este utila si cum se foloseste aceasta tehnica. Si daca vrei mai mult? Poate vrei sa apara link-urile de navigare in lb. romana...sau poate in loc de titlul :"_Index_" ai vrea sa apara "_Lista de Produse_".

Mai jos sunt doua View-uri generate prin Scaffolding si, cu rosu, modificarile dorite de mine:

![](/assets/images/2011/ScaffoldIndexView.png)

![](/assets/images/2011/ScaffoldCreateView.png)

Desigur, aceste modificari se pot face pe fiecare view in parte, dar atunci cand ai 8-10 entitati si 5 forme/entitate, aceasta abordare devine neproductiva.

**Solutia** este sa modifici sursa (template-ul) pe care se bazeaza operatiunea de scaffolding. In realitate sunt mai multe fisiere, cate unul pt. fiecare controller/view. Pt. a vedea locatia originala a acestor fisiere si unde anume trebuie copiate, citeste [articolul lui Hanselman](http://www.hanselman.com/blog/ModifyingTheDefaultCodeGenerationscaffoldingTemplatesInASPNETMVC.aspx).

Pe scurt, copiaza intreg folder-ul `CodeTemplates` de la locatia `\Program Files (x86)\Microsoft Visual Studio 10.0\Common7\IDE\ItemTemplates\CSharp\Web\MVC 3`, in radacina proiectului.

Poti edita fisierele .tt cu orice editor de text, dar pt. a beneficia de Intellisense, Syntax Highlighting etc, atunci foloseste un editor de text specializat. Plugin-ul celor de la [Devart](http://www.devart.com/t4-editor/) permite editarea chiar din VS2010:

![](/assets/images/2011/T4Editor.png)

Pentru a pastra o consistenta la nivelul aplicatiilor pe care le scriu, toate tipurile, proprietatile si metodele din codul sursa, inclusiv coloanele din SQL, le denumesc in lb. engleza. La nivel de interfata, totul vreau sa fie in lb. romana.

Denumirile (in lb. romana) pt. o anumita clasa am ales sa le memorez la nivelul fiecarei clase, sub forma unor metode statice. Clasa "Product" (din [exemplul lui ScottGu](http://weblogs.asp.net/scottgu/archive/2011/05/05/ef-code-first-and-data-scaffolding-with-the-asp-net-mvc-3-tools-update.aspx)) devine:

```csharp
using System;
using System.Collections.Generic;
using System.Linq;
using System.Web;
using System.ComponentModel.DataAnnotations;

namespace Scaffolding.Models
{
	public class Product
	{
		public int ID { get; set; }

		[Display(Name="Nume")]
		public string Name { get; set; }

		[Display(Name = "Categorie1")]
		public int CategoryID { get; set; } //FK property

		[Display(Name = "Pret Unitar")]
		public Decimal? UnitPrice { get; set; }

		[Display(Name = "In Stoc")]
		public int UnitsInStok { get; set; }

		[Display(Name = "Categorie2")]
		public virtual Category Category { get; set; } //Navigation Property

		#region Public Static Methods

		/// <summary>
		/// Folosit in T4 pt. a genera, in View, link-uri de forma "Adauga [Name]"
		/// </summary>
		/// <returns>Numele entitatii, in lb. RO</returns>
		public static string GetEntityName()
		{
			return "Produs";
		}

		/// <summary>
		/// Folosit in T4 pt. a genera, in View, link-uri de forma "Lista de [PluralName]"
		/// </summary>
		/// <returns>Numele entitatii, la plural, in lb. RO</returns>
		public static string GetEntityPluralName()
		{
			return "Produse";
		}

		#endregion
	}
}
```

Sa trecem acum la modificarea fisierelor de configurare:

## 1. Modificari in List.tt (pt. View-ul `Index`)

- Modificarea sectiunii `<title>` (se modifica in 2 locuri, pt. cazurile cu MasterPage sau fara)

      ```
      old: ViewBag.Title = "<#= mvcHost.ViewName#>";
      new: ViewBag.Title = "Lista de " + <#= mvcHost.ViewDataTypeName + ".GetEntityPluralName()" #>;

      old: <title><#= mvcHost.ViewName #></title>
      new: <title>Lista de @<#= mvcHost.ViewDataTypeName + ".GetEntityPluralName()" #></title>
      ```

  Se observa ca sintaxa pt. cele 2 cazuri este diferita.
  In primul caz este vorba de o variabila de tip string (intre ghilimele) care, la runtime, se executa ca si cod C#.
  Acest lucru il stabileste engine-ul Razor in momentul in care intalneste cod intre @{...}. Vezi poza de mai jos:

![](/assets/images/2011/CodeInsideT4.png)

In al 2-lea caz, este vorba de un text direct (fara ghilimele), care, la runtime, va fi randat ca si cod specific HTML

- Modificarea sectiunii `<h2>`: (exista doar pt. cazul cu Master Page)

  ```
  old: <h2><#= mvcHost.ViewName#></h2>
  new: <h2>Lista de @<#= mvcHost.ViewDataTypeName + ".GetEntityPluralName()" #></h2>
  ```

- Modificarea Link-ului `Create New`

  ```
  old: @Html.ActionLink("Create New", "Create")
  new: @Html.ActionLink("Creaza " + <#= mvcHost.ViewDataTypeName + ".GetEntityName()" #>, "Create")
  ```

- Modificarea Link-urilor `Edit` | `Detail` | `Delete`

  ```
  old: @Html.ActionLink("Edit", "Edit", new { id=item.<#= pkName #> }) |
  new: @Html.ActionLink("Editeaza", "Edit", new { id=item.<#= pkName #> }) |
  ```

Idem pt. `Detalii` si `Sterge`. Modifica ambele ramuri ale lui "if" si "else".

- Modificare denumirilor pt. coloanele din grid

  ```
  old: <th> <#= property.AssociationName #> </th>
  new: <th> @Html.LabelFor(x=>x.FirstOrDefault().<#= property.AssociationName #>) </th>
  ```

Obs:

- `property.AssociationName` -> afiseaza [DisplayAttribute] aferenta proprietati virtuale Category din clasa Product
- `property.AssociationNameId` -> afiseaza [DisplayAttribute] aferenta proprietatii CategoryId (de tip guid) din clasa Product

Dupa toate aceste modificari, rezultatul operatiunii de scaffolding va fi urmatorul:

![](/assets/images/2011/ScaffoldIdxOk.png)

## 2. Modificari in Create.tt (pt. View-ul `Create`)

- Modificarea sectiunii `<title>` (se modifica in 2 locuri, pt. cazurile cu MasterPage sau fara)

  ```
  old: ViewBag.Title = "<#= mvcHost.ViewName#>";
  new: ViewBag.Title = "Creaza " + <#= mvcHost.ViewDataTypeName + ".GetEntityName()" #>;

  old: <title><#= mvcHost.ViewName #></title>
  new: <title>Creaza @<#= mvcHost.ViewDataTypeName + ".GetEntityName()" #></title>
  ```

- Modificarea sectiunii `<h2>`

  ```
  old: <h2><#= mvcHost.ViewName#></h2>
  new: <h2>Creaza @<#= mvcHost.ViewDataTypeName + ".GetEntityName()" #></h2>
  ```

- Modificarea sectiunii `<legend>`

  ```
  old: <legend><#= mvcHost.ViewDataType.Name #></legend>
  new: <legend>@<#= mvcHost.ViewDataTypeName + ".GetEntityName()" #></legend>
  ```

- Modificarea denumirilor pt. campurile aferente unei entitati asociate (FK):

  ```
  old: @Html.LabelFor(model => model.<#= property.Name #>, "<#= property.AssociationName #>")
  new: @Html.LabelFor(model => model.<#= property.AssociationName #>)
  ```

- Modificare butonului `Create`

  ```
  old: <input type="submit" value="Create" />
  new: <input type="submit" value="Creaza" />
  ```

- Modificare Link-ului `Back to List`

  ```
  old: @Html.ActionLink("Back to List", "Index")
  new: @Html.ActionLink("Lista de " + <#= mvcHost.ViewDataTypeName + ".GetEntityPluralName()" #>, "Index")
  ```

Dupa toate aceste modificari, rezultatul operatiunii de scaffolding va fi urmatorul :

![](/assets/images/2011/ScaffoldCreateOk.png)

Restul template-urilor (Edit.tt, Delete.tt, Details.tt) se customizeaza dupa modelul lui Create.tt.

**Alte informatii:** [Aici](http://matt-ward.blogspot.ro/2011/10/asp-net-mvc-3-t4-template-properties.html) este lista cu toate proprietatile folosite de T4 Template iar [aici](http://blogs.msdn.com/b/webdev/archive/2009/01/29/t4-templates-a-quick-start-guide-for-asp-net-mvc-developers.aspx) este un ghid de utilizare. Succes!
