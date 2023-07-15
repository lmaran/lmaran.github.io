---
layout: post
title: "Configurarea Disqus in Github Pages"
date: 2014-01-09 00:00:01
comments: true
categories: jekyll
---

[Disqus](http://disqus.com/) este una dintre cele mai cunoscute platforme de "Comments". In continuare am sa amintesc setarile facute de mine pt. a integra aceasta platforma cu [blog-ul personal](http://maran.ro).

In continuare voi presupune urmatoarele:

- site-ul este deja functional si ruleaza cu [Jekyll](http://jekyllrb.com/), in [Github Pages](http://pages.github.com/)
- site-ul este configurat cu custom domain = "http://maran.ro"
- [**important**] serviciul de DNS a fost configurat iar modificarile au avut timp suficient sa se propage (ex:24h). Detalii despre configurarea _DNS_ si _Custom Domain_ am scris [aici](http://maran.ro/2014/01/08/custom-domain-in-github-pages/).

## Configurari pe site-ul Disqus.com

- login pe [http://disqus.com](http://disqus.com)
- creaza un site nou:
  - alege un nume pe post de identificator unic (Ex: **lmaran**)
  - restul setarilor pot ramane nemodificate --> Finish

## Configurari in blog (Jekyll)

- adauga in fisierul `_config.yml` inregistrarile:

```
url: http://maran.ro

disqus_short_name: lmaran
disqus_show_comment_count: true
```

    - ultima setare este optionala
    - **important** este ca `disqus_short_name` sa fie identic cu identificatorul site-ului creat in Disqus

- adauga la sfarsitul fisierului `\_layouts\post.html` urmatorul cod (nemodificat):

```javascript
{% if site.disqus_short_name and page.comments == true %}
<section id="comment">
	<div id="disqus_thread"></div>

   	<script type="text/javascript">
       	var disqus_shortname = '{{ site.disqus_short_name }}';
	        var disqus_identifier = '{{ site.url }}{{ page.url }}';
	        var disqus_url = '{{ site.url }}{{ page.url }}';

		    (function () {
	            var embedScript = document.createElement('script');
	            embedScript.type = 'text/javascript';
	            embedScript.async = true;
	            embedScript.src = 'http://' + disqus_shortname + '.disqus.com/embed.js';
	            (document.getElementsByTagName('head')[0] || document.getElementsByTagName('body')[0]).appendChild(embedScript);

		        var countScript = document.createElement('script');
		        countScript.type = 'text/javascript';
		        countScript.async = true;
		        countScript.src = 'http://' + disqus_shortname + '.disqus.com/count.js';
		        (document.getElementsByTagName('head')[0] || document.getElementsByTagName('body')[0]).appendChild(countScript);
		    }());
   	</script>

   <noscript>Please enable JavaScript to view the <a href="http://disqus.com/?ref_noscript">comments powered by Disqus.</a></noscript>
   <a href="http://disqus.com" class="dsq-brlink">comments powered by <span class="logo-disqus">Disqus</span></a>

</section>
{% endif %}
```

- in "header"-ul fiecarui articol ([YAML Front Matter](http://jekyllrb.com/docs/frontmatter/)) adauga optiunea:

```
comments: true
```

## Rezultatul

![](/assets/images/2014/disqus-demo.png)

Acelasi thread de comentarii este vizibil si atunci cand testez site-ul pe masina locala (**localhost**):

![](/assets/images/2014/disqus-demo-localhost.png)

## Observatii finale

- daca in Disqus am ales **lmaran** pe post de identificator unic al site-ului, atunci comentariile vor putea fi moderate la adresa: [http://lmaran.disqus.com](http://lmaran.disqus.com)
- lasa nemodificata (camp gol) sectiunea "_Trusted Domains_" de pe site-ul Disqus.
- am declarat ca si identificator de pagina (**disqus_identifier**) URL-ul articolului. Am avut grija ca URL-ul sa fie identic (folosind variabila `url` declarata in `_config.yml`), chiar si atunci cand site-ul este rulat de pe localhost (Jekyll local). In caz contrar, in sectiunea "Discussions" de pe site-ul Disqus am fi avut acelasi articol de 2 ori, fiecare cu propriul thread de comment-uri:
  - **title**: articol1; **link**: http://localhost:4000/2014/10/01/articol1
  - **title**: articol1; **link**: http://maran.ro/2014/10/01/articol1
- daca stergi din Disqus site-ul cu comentarii aferent blog-ului si apoi il refaci folosind **acelasi identificator de site**, atunci sectiunea de comentarii din blog va redeveni functionala, fara nici o alta modificare. Desigur, vechile comentarii vor disparea.
- detalii despre variabilele **disqus**, [aici](http://help.disqus.com/customer/portal/articles/472098-javascript-configuration-variables)
- detalii despre variabilele **jekyll**, [aici](http://jekyllrb.com/docs/variables/)

**Inspiratie:** setarile Javascript de mai sus reprezinta o combinatie dintre [setarile disqus originale](http://help.disqus.com/customer/portal/articles/472097-universal-embed-code) si setarile preluate de pe [blog-ul lui Phill Haack](https://github.com/Haacked/haacked.com).
