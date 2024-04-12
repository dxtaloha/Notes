## 先决条件

sqlmap需要python环境



## 手册

https://blog.csdn.net/wn314/article/details/78872828

https://github.com/itechub/sqlmap-wiki-zhcn/releases



## 基本用法

-u:扫描某url(GET方式)

-r:扫描某以POST方式提交的http请求（-r目标为.txt文件，POST方式）

-p:指定POST方式下扫描的注入表单参数

-D:已知的库名

--dbs:给出所有数据库

--current-db：给出当前所在数据库

-T:已知的表名

--tables：给出所有表名

-C:已知的列名

--colums：给出所有列名

--dump:加载所有当前列的数据 //尽量不用，会下载数据

-m txt.txt批量扫描url（不加-u）

总体上以上的用法即先看有没有注入点，然后--dbs爆数据，--current-db爆当前数据库，-D xxx --tables爆表，-D xxx -T xxx --colums爆列，-D xxx -T xxx --C xxx,xxx,xxx --dump爆具体数据。