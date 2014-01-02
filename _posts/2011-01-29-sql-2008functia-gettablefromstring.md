---
layout: post
title:  "SQL 2008: functia GetTableFromString (Google Style)"
date:   2011-01-29 00:00:02
comments: true
categories: MSSQL
---

Functia de mai jos returneaza un tabel in care, pe o coloana sunt depozitate toate cuvintele incluse intr-un string acceptat ca parametru. Cealalta coloana este de tip Integer- auto si poate fi utila atunci cand este decesara apelearea cuvintelor prin indexul lor (ex. apelare in bucla)

Functia de mai jos returneaza un tabel in care, pe o coloana sunt depozitate toate cuvintele incluse intr-un string acceptat ca parametru. Cealalta coloana este de tip Integer- auto si poate fi utila atunci cand este decesara apelearea cuvintelor prin indexul lor (ex. apelare in bucla)

Exemplu de apel cu spatiu pe post de ch. de delimitare:

![](https://dl.dropboxusercontent.com/u/43065769/blog/images/2011/GetTbl.png)

**Codul**:

    CREATE FUNCTION [dbo].[_GetTableFromString]
    (@InputString NVARCHAR(max), @SplitChar CHAR(1))
    RETURNS @ValuesList TABLE
    	(
    		id INT IDENTITY(1,1),
    		param NVARCHAR(255)
    	)
    AS
    
    BEGIN
    	DECLARE @ListValue NVARCHAR(max)
    	SET @InputString = ltrim(rtrim(@InputString))+ @SplitChar
    	WHILE len(@InputString)>0 and CHARINDEX(@SplitChar, @InputString)>0
    	--inseamna ca am cel putin un cuvant si acesta este urmat de un spatiu
    	BEGIN
    		--extrag primul cuvant
    		SELECT @ListValue = SUBSTRING(@InputString , 1, CHARINDEX(@SplitChar, @InputString)-1)
    		--adauga cuvantul in tabela
    		INSERT INTO @ValuesList	SELECT @ListValue
    		--reinitializez sirul eliminand primul cavant
    		SELECT @InputString = ltrim(SUBSTRING(@InputString, len(@listValue)+1, len(@InputString)-len(@listValue)+1))
    	END
    RETURN
    END

[Aici](http://maran.ro/2011/01/30/sql-2008-procedura-filtertablebystring/) este un exemplu in care folosesc aceasta functie. 