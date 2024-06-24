##### 1、grep

正则表达式搜索

```bash
grep ^fu.*ck$ xxx
```

代表在文件中找到以fu开头(^)和ck结尾($)的行，且fu和ck中间可以有任意数量的任意字符(.*)

##### 2、uniq

用于报告或忽略重复的行（它只报告相邻重复的行，而并不报告不相邻但有重复的行）

```bash
#-c在每行前面显示重复的次数
#-d只显示重复的行
#-u只显示不重复的行
#-i忽略大小写
#-w N：比较每行的前N个字符
#-f N：忽略每行前N个字段

#去除相邻重复的行
uniq xxx 
#只显示重复的行
uniq -d xxx
```

##### 3、sed

用于搜索、编辑、删除文件中的内容

```bash
#-n抑制输出，只输出指定的内容
#22p的p为print，22d的d代表delete

#删除
sed -n '22d' xxx
#打印
sed -n '22p' xxx
```

##### 4、sort

用来给一个文件内容排序，没有参数即默认基于ASCII字符集排序

```bash
#-b忽略每行前的空白字符
#-f忽略大小写
#-h按人类可读的顺序排序（2K，1G）
#-n按数值大小排序
#-d使用字典排序（仅考虑字母、数字和空白）
#-u去重排序
#-k指定按第几个字段排序（即按第几列排序，列之间用空格隔开）
#-r逆排序

#默认排序并在当前终端打印出来
sort xxx |grep 1
```

##### 5、find

用于查找系统中的文件

```bash
#type按类型查找，f是filename文件，d是direction目录
find / -type f/d
#name按名查找
find / -name xxx
#mtime按修改时间查找，+为查找3214天前修改的文件，-3214查找3214天前到现在期间修改的文件，没有后+—代表查找3214天前这天修改的文件
find / -mtime /+/-3214
#size按大小查找，c代表字节，k千字节，M兆字节，G吉字节，+代表查找比该它大的，-代表查找比它小的
find / -size /+/-3243c/k/M/G
#按文件所有者查找
find / -user xxx
#exec，对查找的文件执行某个命令
find / -name xxx -exec cat yyy {} \;
```

##### 6、cat

cat的对象不能是-之类的特殊字符，得加./-才能cat

##### 7、2>/dev/null

将标准错误抛弃不显示

##### 8、>、&、<、$？

&存在是为了告诉bash接下来是一个文件描述符

$?为最近执行命令的退出状态，0为成功执行，其它数位不成功执行，不同值为不同类型错误

```bash 
#将错误输出重定向到标准输出
cat xxx 2>&1
#将错误输出重定向到文件1中
cat xxx 2>1

#先用fasle使得$?为0，因为接着执行了echo，所以echo的结果为非0，此后再执行一个echo则为0
fasle; echo $?; echo $?
```

##### 8、file

识别文件类型

```bash
file xxx
#如果使用管道时应该为
|file -
```

##### 9、base64

base64编解码

```bash
#-d解码，-e编码
base64 -d/-e xxx
```

##### 10、xxd

转换进制

```bash
#将xxx转换为十六进制和ascii表示
xxd xxx
#将十六进制转换为二进制表示
xxd -r xxx
```

##### 11、read

从标准输入中读取一行并将其赋值给某个变量

```bash
#-r禁止反斜杠转义字符（保留原始输入）
#标准输入中读取赋值给a
read -r a
#从文件中读取（替换标准输入）赋值给a
read -r a < file
```

```bash
#readlink
```

##### 12、readlink

返回文件的绝对路径

```bash
#-e返回存在文件的绝对路径
readlink -e /path/to/file

#可以和read联合使用，假设把标准输入赋值给了$a,可以配合while判断标准输入的所有输入的路径是否存在
readlink -e /path/to/file/$a


```

##### 13、IFS

内部字段分隔符

```bash
#当为空时，每一行会作为一个整体被读取
IFS= read -r a
#当为空格、制表符、换行符时，IFS会把每一行按空格、制表符、换行符分隔为多个整体被读取
IFS=' ' read -r a
IFS=$'\t' read -r a
IFS=$'\n' read -r a
```

##### 14、while

循环

```bash
while condition; do command; done
```

```bash
#while循环提取一个文件中的每行数据赋值给变量以供使用
while IFS= read -r line; do readlink -e /etc/xdg/$line ; done < dict.txt
#while循环不停轮询一个文件中是否拥有内容并直到该文件存在内容为止
false; while [ $? -ne 0 ]; do cat /free/* ; done 2>/dev/null
```

##### 15、ssh

```bash
#创建一个SSH隧道，将本地端口转发到远程主机上的指定端口
#该命令执行结果是：利用ssh连接到远程主机的5000端口，并将本地的9001端口收到的数据转发到远程主机的本地地址上的80端口
#lola@venus.hackmyvm.eu是远程主机，127.0.0.1:80是要转发到远程主机的某个地址的某个端口（这里是远程主机自己的本地地址）,127.0.0.1:9001是本地主机的哪个地址的哪个端口（这里是本地主机的本地地址，一般可以省略，默认为本地地址）
ssh -L 127.0.0.1:9001:127.0.0.1:80 lola@venus.hackmyvm.eu -p 5000
```

