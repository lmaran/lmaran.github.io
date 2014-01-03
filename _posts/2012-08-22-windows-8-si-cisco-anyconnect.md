---
layout: post
title:  "Windows 8 si Cisco AnyConnect"
date:   2012-07-15 00:00:01
comments: true
categories: IT-infrastructure, Windows-8
---

Pe sistemele de operare anterioare, instalarea si conectarea cu clientul de VPN de la Cisco se realizeaza fara probleme. Pe Windows 8, instalarea se face ok, dar apare o problema in momentul stabilirii conexiunii. Mai precis, mesajul de **eroare** este urmatorul:
"*AnyConnect was not able to establish a connection to the specified secure gateway.*"

![](https://dl.dropboxusercontent.com/u/43065769/blog/images/2012/1AnyConnectErr.png)

**Cauza** problemei consta in faptul ca, pentru securizarea conexiunii, server-ul de VPN emite un certificat semnat de o autoritate nerecunoscuta de catre sistemul de operare. In cazul de fata, certificatul este "*self signed*".

**Solutia** presupune identificarea si instalarea certificatului pe masina cu Windows 8. Cum?

Daca ai putina dexteritate, poti apela direct la varianta scurta, prezentata de Bobby J Cannon, [aici](http://social.technet.microsoft.com/Forums/en-US/a116ddd0-c31d-4bd3-a5a6-91e1b6d4dafa/cisco-anyconnect-vpn-and-windows-8?forum=W8ITProPreRel). Aceiasi pasi, dar intr-o prezentare "for dummies" ar fi:

1.  Lansezi IE ca si “administrator“:

 ![](https://dl.dropboxusercontent.com/u/43065769/blog/images/2012/0AnyConnectStartIE.png)

2.  In browser, introduci numele server-ului de VPN (cu https):

 ![](https://dl.dropboxusercontent.com/u/43065769/blog/images/2012/2AnyConnectSite.png)

3.  Click pe icoana rosie din URL pentru a accesa, si apoi pentru a instala certificatul (daca nu ai disponibil butonul de "Install", atunci, f.probabil, nu ai rulat IE ca si "administrator" – v. pas 1)

 ![](https://dl.dropboxusercontent.com/u/43065769/blog/images/2012/4AnyConnectInstallCertif.png)

4. Instaleaza in locatia aferenta lui "Current User":

 ![](https://dl.dropboxusercontent.com/u/43065769/blog/images/2012/5AnyConnectInstallCertif.png)

5. ...si, neaparat, in "*Trusted Root Certification Authorities*":

 ![](https://dl.dropboxusercontent.com/u/43065769/blog/images/2012/5AnyConnectPathCertif.png)

6. Next, Next, Next:

 ![](https://dl.dropboxusercontent.com/u/43065769/blog/images/2012/6AnyConnectFinishCertif.png)

 ![](https://dl.dropboxusercontent.com/u/43065769/blog/images/2012/7AnyConnectWarningCertif.png)

 ![](https://dl.dropboxusercontent.com/u/43065769/blog/images/2012/8AnyConnectComplete.png)


In acest moment, conexiunea poate fi stabilita:

 ![](https://dl.dropboxusercontent.com/u/43065769/blog/images/2012/10AnyConnectTestOK.png)

La mine a functionat. Asta e tot!