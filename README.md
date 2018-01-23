# MS-SQL-SERVER
* 查询数据库中的所有数据库名： <br>
SELECT Name FROM Master..SysDatabases ORDER BY Name <br>
* 查询某个数据库中所有的表名： <br>
SELECT Name FROM SysObjects Where XType='U' ORDER BY Name <br>
* 查询表结构信息：<br>
```sql
SELECT d.NAME                                           AS TableName, 
       A.colorder, 
       A.NAME                                           AS ColName, 
       ( CASE 
           WHEN Columnproperty(a.id, a.NAME, 'IsIdentity') = 1 THEN 1 
           ELSE 0 
         END )                                          IsIdentity, 
       ( CASE 
           WHEN (SELECT Count(1) 
                 FROM   sysobjects 
                 WHERE  ( NAME IN (SELECT NAME 
                                   FROM   sysindexes 
                                   WHERE  ( id = a.id ) 
                                          AND ( indid IN (SELECT indid 
                                                          FROM   sysindexkeys 
                                                          WHERE  ( id = a.id ) 
                                                                 AND ( colid IN 
                                                                     (SELECT 
                                                                       colid 
                                                                     FROM 
                                                         syscolumns 
                                                                   WHERE 
                                                                     ( 
                                                         id = a.id ) 
                                                         AND ( 
                                                                    NAME = a.NAME )) )) )) ) 
                        AND ( xtype = 'PK' )) > 0 THEN 1 
           ELSE 0 
         END )                                          IsKey, 
       B.NAME                                           AS ColDataType, 
       -- 数据类型 
       A.length                                         AS Bytes, 
       -- 占用存储空间 
       Columnproperty(a.id, a.NAME, 'PRECISION')        AS Length, 
       -- 字符串长度 
       Isnull(Columnproperty(a.id, a.NAME, 'Scale'), 0) AS ScaleSize, 
       -- 小数位数 
       A.isnullable,-- 允许空 
       Isnull(e.text, '')                               AS DefaultValue, 
       Isnull(g.[value], ' ')                           AS Remark 
FROM   syscolumns A 
       LEFT JOIN systypes B 
              ON a.xtype = b.xusertype 
       INNER JOIN sysobjects D 
               ON a.id = d.id 
                  AND d.xtype = 'U' 
                  AND d.NAME <> 'dtproperties' 
       LEFT JOIN syscomments E 
              ON a.cdefault = e.id 
       LEFT JOIN sys.extended_properties G 
              ON a.id = g.major_id 
                 AND a.colid = g.minor_id 
       LEFT JOIN sys.extended_properties F 
              ON d.id = f.class 
                 AND f.minor_id = 0 
WHERE  B.NAME IS NOT NULL 
--WHERE d.name='要查询的表' --如果只查询指定表,加上此条件 
ORDER  BY d.NAME, 
          A.id, 
          a.colorder 
```
* 截取&左边的所有字符：<br>
```sql
SELECT LEFT(Cast(actions_name AS VARCHAR(255)), (Charindex('&', Cast(actions_name AS VARCHAR(255))) - 1 )) 
FROM   ceair_shopping.dbo.shopping_actions 
WHERE  actions_name LIKE 'shopping.ceair.com/search?category3_id=%' 
       AND actions_name NOT LIKE 'shopping.ceair.com/search?category3_id=&%' 
       AND Charindex('&', Cast(actions_name AS VARCHAR(200))) > 0 
```

*  显示数据库 & 表的size
```sql
EXEC sp_spaceused
EXEC sp_spaceused 'table_name'
 ```
列名	数据类型	描述<br>
database_name	varchar(18)	当前数据库的名称。<br>
database_size	varchar(18)	当前数据库的大小。<br>
unallocated space	varchar(18)	数据库的未分配空间。<br>

列名	数据类型	描述<br>
reserved	varchar(18)	保留的空间总量。<br>
Data	varchar(18)	数据使用的空间总量。<br>
index_size	varchar(18)	索引使用的空间。<br>
Unused	varchar(18)	未用的空间量。<br>
