---
title: mysql中delete的语法别名问题
date: 2018-6-10 20:44:05
tags: 
	- mysql
categories: 
	- mysql	
---

首先确认，mysql中的delete语句是支持别名的；

在自己书写delete语法时候，语句如下：

```sql
delete from tableA a where a.c_pk_id = '123'
```

但是会报一个别名使用错误，如下：

```sql
[Err] 1064 - You have an error in your SQL syntax; check the manual that corresponds to your MySQL server version for the right syntax to use near 'q  
WHERE  
    q.C_PLY_NO = '1100107000404000220150000001'  
AND q.N_EDR_PRJ_NO = '1'' at line 3  
```

通过查询资料得知，mysql的delete的语法有些特殊，如下：

```sql
delete   a   from tableA  a  where a.c_pk_id = '123'

```
成功删除！！！
比较之后可知道，delete语句在用别名的时候要多写一个别名在delete后边


延伸：

（ORACLE适用）    DELETE FROM TABLEA A WHERE A.FIELD1=10
（SQLSERVER适用） DELETE TABLEA FROM TABLEA A WHERE A.FIELD1=10
（Ora\SQL均适用） DELETE FROM TABLEA WHERE TABLEA.FIELD1=10

