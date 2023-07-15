---
layout: post
title: "EF4 Code First CTP5 si SQL Azure"
date: 2011-01-29 00:00:01
comments: true
categories: SQLAzure EF
---

**UPDATE** (25.09.2012) – Codul aferent versiunii EF5 este [aici](http://pastebin.com/688DhJXz). Evident, in codul atasat trebuie actualizate valorile pt. conexiunea la SQL Azure: _mysrvname_, _myuser_, _mypassword_.

Abordarile **Database First** si **Model First** pe care le suporta Entity Framework sunt compatibile de ceva vreme cu SQL Azure. Fiindca pana in momentul in care scriu acest articol nu am gasit nici un exemplu care sa-mi confirme daca si recenta abordare Code First lucreaza cu SQL Azure, am decis sa testez singur acest lucru.

Din VS2010 am creat o 'Console Application'. Codul este simplist: 2 clase POCO care modeleaza entitatile _Elevi_ si _Clase_ si apoi clasa _ScoalaContext_ care face legatura cu baza de date. Fiindca pluralizarea se face automat dupa regulile gramaticii engleze, am preferat sa stabilesc manual numele tabelelor. Desi nu era necesar, am inlocuit si schema implicita (_dbo_) cu una proprie (_MySchema_). Pt. a fi mai usor de urmarit codul, Connection String-ul (CN) l-am scris alaturi de restul codului si i-am adaugat atributul `MultipleActiveResultSets=True`. In final, la sectiunea principala am adaugat cativa elevi (si clasele din care fac parte) pe care ulterior ii afisez.

## Etapele parcurse pt. testare:

1. Verific situatia actuala din cloud si ma asigur ca nu exista nici o DB.

![](/assets/images/2011/SqlAzure1.png)

2. Scriu si rulez codul (vezi la final).

3. Rezultatul de pe ecran la prima rulare:

![](/assets/images/2011/3Elevi.png)

4. Confirm ca datele au fost stocate in SQL Azure. La prima rulare s-au creat DB, schema, tabelele aferente entitatilor, tabela _EdmMetadata_ etc

![](/assets/images/2011/DBAzure.png)

5. Rulez programul a 2-a oara. Infrastructura se pastreaza iar datele se adauga la cele existente:

![](/assets/images/2011/6Elevi1.png)

6. Datele:

![](/assets/images/2011/TblElevi.png)

7. ...si codul (totul este intr-un singur fisier):

```csharp
using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;
using System.Data.Entity;
using System.Data.Entity.Database; //pt. DbDatabase
using System.Data.Entity.ModelConfiguration; //Pt. ModelBuilder

namespace CodeFirstSample
{
    public class Clasa
    {
        public int ClasaId { get; set; }
        public string NumeClasa { get; set; }
        public virtual ICollection Elevi { get; set; }
    }

    public class Elev
    {
        public int ElevId { get; set; }
        public string Nume { get; set; }
        public string Prenume { get; set; }
        public int ClasaId { get; set; }
        public virtual Clasa Clasa { get; set; }
    }

    public class ScoalaContext : DbContext
    {
        public ScoalaContext()
            : base("Server=tcp:nlqbii4xj3.database.windows.net;" +
            "Database=MyAzureDb;" +
            "User ID=xxxxx@aaaaa;Password=yyyyy;" +
            "Trusted_Connection=False;Encrypt=True;" +
            "MultipleActiveResultSets=True") // optiune necesara doar pt. SQL Azure
        { }

        public DbSet Clase { get; set; }
        public DbSet Elevi { get; set; }

        protected override void OnModelCreating(ModelBuilder modelBuilder) //Optional
        {
            modelBuilder.Entity()
                .Map(m => m.ToTable("Elevi", "MySchema"))
                .Property(m => m.ClasaId).IsOptional()
                ;

            modelBuilder.Entity()
                .Map(m => m.ToTable("Clase", "MySchema"));
        }
    }

    class Program
    {
        static void Main(string[] args)
        {
            using (var db = new ScoalaContext())
            {
                //Insereaza date de test in DB
                var clasa1 = new Clasa { NumeClasa = "Clasa I-a" };
                var clasa2 = new Clasa { NumeClasa = "Clasa a II-a" };

                db.Elevi.Add(new Elev
                {
                    Nume = "Popescu",
                    Prenume = "Gigel",
                    Clasa = clasa2
                });

                db.Elevi.Add(new Elev
                {
                    Nume = "Ionescu",
                    Prenume = "Ana",
                    Clasa = clasa1
                });

                db.Elevi.Add(new Elev
                {
                    Nume = "Florescu",
                    Prenume = "Ioana",
                    Clasa = clasa2
                });
                db.SaveChanges();

                //Afiseaza datele din DB
                var totiElevii = from p in db.Elevi select p;
                Console.WriteLine("Elevi pe clase:");
                foreach (var elev in totiElevii)
                    Console.WriteLine(" - {0} {1}, {2}, {3}", elev.ElevId, elev.Nume, elev.Prenume, elev.Clasa.NumeClasa);
            }
        }
    }

}
```
