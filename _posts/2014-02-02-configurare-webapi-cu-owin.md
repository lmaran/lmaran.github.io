---
layout: post
title: "Configurare WebApi cu OWIN"
date: 2014-02-02 00:00:01
comments: true
categories: WebApi Owin
---

Voi pleca de la un proiect complet gol.
Voi adauga apoi bibliotecile si setarile **minim** necesare a.i proiectul de tip WebApi obtinut sa functioneze ca o suma de module interconectate dupa specificatiile [OWIN](http://owin.org/).

Codul de mai jos e disponibil si pe [github](https://github.com/lmaran/DemoOwin/tree/master/InstallWebApiOwinIIS).

## "Hello world" - doar cu OWIN

1. adauga un proiect gol

![](/assets/images/2014/add-empty-project.png)

2. Instaleaza unul din cele 3 **host**-uri care va gazdui aplicatia:

- `Install-Package Microsoft.Owin.Host.SystemWeb` - (cu IIS si ASP.NET)
- `Install-Package Microsoft.Owin.Host.HttpListener` - "SelfHost" (fara "IIS" si fara ASP.NET)
- `Install-Package Microsoft.Owin.Host.IIS -pre` - "Helios" (cu "IIS" dar fara ASP.NET)

Actualizeaza pachetele instalate. Cel putin pt. varianta actuala de "Helio" acest pas este obligatoriu:

- `Update-Package -pre`

Recomand varianta Helios. Detalii [aici](http://maran.ro/2014/06/06/helios-versus-systemweb/).

Dll-urile referentiate pana acum (legate de OWIN cu System.Web) ar fi urmatoarele:

![](/assets/images/2014/owin-references.png)

3. Creaza in radacina proiectului un fisier numit `Startup.cs` cu urmatorul continut:

   ```csharp
   using Owin;

   namespace Web
   {
       public class Startup
       {
           public void Configuration(IAppBuilder app)
           {
               app.Run(context =>
               {
                   context.Response.ContentType = "text/plain";
                   return context.Response.WriteAsync("Hello World!");
               });
           }
       }
   }
   ```

...sau, acelasi lucru in varianta "async"

    ```csharp
    ...
    app.Run(async context =>
    {
        context.Response.ContentType = "text/plain";
        await context.Response.WriteAsync("Hello World!");
    });
    ...
    ```

4. test OWIN (fara WebApi):

![](/assets/images/2014/test-owin-only-ok.png)

## "Hello world" - OWIN si WebApi

1. Acum, ca host-ul de tip OWIN am vazut ca functioneaza, urmeaza sa "prelungesc" pipeline-ul a.i. cererea sa fie servita de un controller WebApi.

`Install-Package Microsoft.AspNet.WebApi.Owin -pre`.

2.  Modific fisierul `Startup.cs` astfel:

    ````csharp
    public class Startup
    {
    public void Configuration(IAppBuilder app)
    {
    var config = new HttpConfiguration();
    WebApiConfig.Register(config);

                app.UseWebApi(config);
            }
        }
        ```

    unde `WebApiConfig` este un fisier in care am izolat setarile WebApi, astfel:

        ```csharp
        public static class WebApiConfig
        {
            public static void Register(HttpConfiguration config)
            {
                // Web API routes
                config.MapHttpAttributeRoutes();

                config.Routes.MapHttpRoute(
                   name: "Home",
        		   //routeTemplate: "{*anything}",
                   routeTemplate: "{controller}",
                   defaults: new { controller = "Home" });
            }
        }
        ```

    ````

3.  Apoi, am creat folder-ul Controllers in care am adaugat un controller simplu, ca mai jos:

    ```csharp
    public class HomeController : ApiController
    {
        public string Get()
        {
            return "hello world";
        }
    }
    ```

4.  In acest moment structura proiectului este urmatoarea:

![](/assets/images/2014/webapi-owin-files-config1.png)

5. iar la rularea aplicatiei ar trebui sa primim mesajul "hello world", returnat de controller:

![](/assets/images/2014/webapi-response-ok1.png)

## Pregatirea WebApi pentru SPA

Adica la start (in HomeController) voi returna un fisier static (index.html) iar restul controller-elor le voi pregati sa raspunda la cereri API de tip REST.

1. adauga in `WebApiConfig` o ruta pt. apelurile de tip API:

   ```csharp
   public static class WebApiConfig
   {
       public static void Register(HttpConfiguration config)
       {
           // Web API routes
           config.MapHttpAttributeRoutes();

           config.Routes.MapHttpRoute(
               name: "DefaultApi",
               routeTemplate: "api/{controller}/{id}",
               defaults: new { id = RouteParameter.Optional }
           );

           config.Routes.MapHttpRoute(
              name: "Home",
              //routeTemplate: "{*anything}",
              routeTemplate: "{controller}",
              defaults: new { controller = "Home" });
       }
   }
   ```

2. adauga un controller nou, de test:

   ```csharp
   public class ValuesController : ApiController
   {
       public IEnumerable<string> Get()
       {
           return new string[] { "value1", "value2" };
       }
   }
   ```

3. test:

![](/assets/images/2014/webapi-response-ok2.png)

4. adaug in radacina proiectului un fisier simplu, de tip html:

   ```html
   <!DOCTYPE html>
   <html xmlns="http://www.w3.org/1999/xhtml">
     <head>
       <title></title>
     </head>
     <body>
       Hello world (inside index.html file)
     </body>
   </html>
   ```

5. iar in `HomeController` modific metoda `Get()` astfel:

   ```csharp
   public HttpResponseMessage Get(){
       var filePath = HostingEnvironment.MapPath("~/index.html");

       var response = new HttpResponseMessage(HttpStatusCode.OK);
       response.Content = new StringContent(File.ReadAllText(filePath));
       response.Content.Headers.ContentType = new MediaTypeHeaderValue("text/html");
       return response;
   }
   ```

6. test (se observa ca numele `index.html` nu apare in URL):

![](/assets/images/2014/webapi-response-ok3.png)
