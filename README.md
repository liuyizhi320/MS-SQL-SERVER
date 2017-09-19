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
