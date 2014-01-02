---
layout: post
title:  "SQL 2008: procedura FilterTableByString (Google Style)"
date:   2011-01-30 00:00:01
comments: true
categories: MSSQL
---

Procedura cauta intr-o coloana a unei tabele dupa mai multe cuvinte introduse in formatul "Google Style". Pt. a mari flexibilitatea, numele tabelei si numele campului in care se face cautarea le-am declarat variabile.

**Exemplu**:

![](https://dl.dropboxusercontent.com/u/43065769/blog/images/2011/lucimar.png)

**Codul:**

    CREATE PROCEDURE [dbo].[_FilterTableByString]
    @CuvinteCautare varchar(100), @Coloana varchar(100), @Tabela varchar(100)
    AS
    BEGIN
    	--SELECT * FROM _GetTableFromString(@CuvinteCautare,' ') --pt. depanare
    	DECLARE @contor int, @contorTotal int, @sql nvarchar(max),@cuvant nvarchar(max)
    
    	SET @sql = N'SELECT * FROM ' + @Tabela
    	SELECT @contorTotal = count(*) from _GetTableFromString(@CuvinteCautare,' ')
    		BEGIN
    
    			SET @contor=1
    			WHILE (@contor<=@contorTotal)
    			BEGIN
    					SET @cuvant = (SELECT [param] FROM _GetTableFromString(@CuvinteCautare,' ') WHERE id=@contor)
    					IF @contor = 1
    						SET @sql = @sql + ' WHERE '
    					ELSE
    						SET @sql = @sql + ' AND '
    
    					SET @sql = @sql + @Coloana + N' LIKE ''%' + @cuvant + '%'''
    					SET @contor = @contor+1
    			END
    		END
    		--PRINT @sql --pt. depanare
    	EXEC sp_executesql @sql
    END
In codul de mai sus am folosit functia _GetTableFromString, implementata [aici](http://maran.ro/2011/01/29/sql-2008functia-gettablefromstring).