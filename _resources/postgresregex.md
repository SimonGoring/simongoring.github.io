---
title: R regex to convert SQL queries
concept: Using regular expressions and R to convert queries in SQL Server to postgres to assist with a database Migration.
layout: page
---

Old function:
```SQL
CREATE PROCEDURE [GetAgeTypesTable]
AS 
SELECT     AgeTypeID, AgeType, Precedence, ShortAgeType
FROM         NDB.AgeTypes






GO
```

needs to look like:
```
CREATE OR REPLACE FUNCTION ti.getagetypestable()
 RETURNS SETOF ndb.agetypes
 LANGUAGE sql
  AS $function$
  SELECT ndb.agetypes.* FROM ndb.agetypes;$function$
```
