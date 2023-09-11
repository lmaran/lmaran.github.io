---
layout: post
title:  "Generatoare de documente PFD"
date:   2023-07-02 21:19:24 +0300
categories: pdf
---

# A. Solutii pt. creat PDF "from scratch"

## pdfkit

- A JavaScript PDF generation library for Node and the browser.
- documentul se creaza la nivel de pixel (low level)
- MIT, 2005 stars, lu=6 days ago
- sta la baza altor librarii precum pdfmake
- I use pdfkit because I have an offline-capable JS app with fairly rich document printing needs. A friendly layout engine on top (pb. se refera aici la pdfmake n.a.) can save a lot of effort making nice designs.

## pdfmake

- Client/server side PDF printing in pure JavaScript
- full declarativ (intreg documentul se defineste printr-un json)
- MIT, 3195 stars, last update=19days
- este de fapt un layout engine peste pdfkit (acesta din urma se ocupa de generarea pdf-ului)
- I used PDFMake on a recent project and was very pleased. One issue we ran into in browser was the huge size overhead from vfs_fonts.


# B. Solutii pt. creat PDF "from html"

OBS: aceste solutii sunt mai putin precise decat variante de generare "from scratch":
They (HTML and PDF n.a.) don't really map 1:1. The PDF exported by a browser from an HTML page is an interpretation of that content. Converting from HTML is not a straightforward, reliable way to produce PDFs. Sometimes the process turns out beautiful, sometimes it turns out kind of shitty.

node-wkHtmlToPdf:
- converteste HTML in PDF folosindu-se de Webkit
- este de fapt un wrapper peste un command-line tool (wkhtmltopdf) care tb. sa fie instalat
- MIT, 137 stars, 9 months ago

phantomjs:
- are capabilitatea de a genera PDF-uri fiindca Phantom in sine este un "headless WebKit" - iar webkit stie asta


# Discutii

- generarea "client-side" are 2 avantaje:
  - iti permite sa lucrezi offline
  - scalabil - nu conteaza cati clienti ai din moment ce procesarea se face pe client.

# Surse

- https://news.ycombinator.com/item?id=9630431
- http://www.feedhenry.com/server-side-pdf-generation-node-js/