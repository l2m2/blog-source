---
title: Sql Server将Image字段转为Base64字符串
toc: false
date: 2020-01-08 08:50:38
description: 你可能不知道的SQL
tags:
- MSSQL
- SQL
---

直接在SQL查询将Image类型的字段转为Base64字符串。

```sql
DECLARE @img1 VARBINARY ( max ) 
SELECT @img1 = ImageCol FROM xxx WHERE col = '12159' 
SELECT *, CAST( '' AS XML ).value( 'xs:base64Binary(sql:variable("@img1"))', 'VARCHAR(MAX)' ) AS img1 
FROM xxx WHERE col = '12159'
```

OR

```sql
select base64
from xxx
cross apply (select ImageCol as '*' for xml path('')) T (base64)
where col = '12159'
```

## Reference

-  https://stackoverflow.com/questions/38324674/convert-image-to-base64string-in-select-query 
-  https://stackoverflow.com/questions/45664937/retrieve-varbinary-value-as-base64-in-mssql 

