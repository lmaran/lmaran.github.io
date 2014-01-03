---
layout: post
title:  "Send email, cu autentificare, din command prompt"
date:   2011-02-09 00:00:01
comments: true
categories: IT-tools
---

De ce ai face asta din command promt? Poate vrei sa testezi conectivitatea cu un server de email de pe o masina care nu are instalat un client de email. Sau poate exista un astfel de client dar chiar acesta trebuie diagnosticat.

Un telnet la serverul si portul de SMTP imi confirma ca am conectivitate, porturi deschise si serviciu disponibil. Daca vreau insa sa testez intreg procesul de trimitere a unui email (conectivitate,autentificare, relay, send data...) atunci trebuie ca, dupa stabilirea conexiunii, sa incep sa "discut" cu seviciul de SMTP "spunand-i" ce anume doresc sa faca pt. mine.

Un astfel de dialog arata astfel:

![](https://dl.dropboxusercontent.com/u/43065769/blog/images/2011/cmd_send_email.png)

**Rezultatul:**

![](https://dl.dropboxusercontent.com/u/43065769/blog/images/2011/rezultatSendEmail.png)

Comenzile introduse de client sunt subliniate cu *albastru* iar raspunsul primit de la server, cu negru. Comenzile sunt comentate cu *verde*. 

Partea mai putin cunoscuta este sectiunea de autentificare, incercuita cu *rosu*. Si nu cred ca exista, in lumea reala, servere de email care sa trimita emailuri pt. utilizatori neautentificati.

Numele si parola nu pot fi introduse in text clar. Ele trebuiesc intai codificate cu algoritmul Base64. Pt. asta am folosit utilitarul `EB64.exe`   disponibil [aici](http://www.petri.co.il/smtp-authentication.htm). Se descarca si se ruleaza direct din cmd, cu sintaxa:

![](https://dl.dropboxusercontent.com/u/43065769/blog/images/2011/codificareBase64.png)

Asta e tot!

Stie cineva cum se poate trimite un email (tot din cmd, evident) folosind un server care accepta doar conexiuni SSL? Eu nu am reusit.