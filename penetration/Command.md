##### bash注意事项

```bash
#''中所有的特殊字符都不会被解析和执行，而且''内不允许包含'，因为会关闭字符串


#""中只有$、\、``三个特殊字符会被解析但不会被执行，其它均不会被解析执行。其中``位反引号，反引号内容会作为命令被执行。""包含"需要使用\来转义
####举例（仅解析的结果就是成为一个值，而不是一个可以被执行的命令；只有解析后被执行，解析的结果才会作为命令被执行）
a="echo kkk"
$a ---> kkk (解析+执行)(解析后作为命令被执行)
echo $a ---> echo kkk (仅解析)(解析后作为值被传递，仅解析。使用$a是解析+执行还是仅解析是解释器自行判断的)
"$a" ---> echo kkk: command not found (仅解析)(解析后为一个值，无法作为命令执行，想要执行它用eval，即eval "$a" ---> kkk)
${a} === $a (完全等价，使用变量的时候尽量用${a})
####


#bash任何命令都不会将端点的""和''作为输出
```

```bash
#变量在赋值时不加$，只有在使用读取值时才加$。而且只有变量是预期输入时会直接作为输入，否则会作为命令被执行
a="echo kkk"
$a ---> kkk (作为命令)
echo $a ---> echo kkk (作为值)
```

```bash
#$0代表命令自身，$1代表命令行第一个参数，以此类推
#$!保存有最近启动的后台进程的PID（一般就是当前进程PID）
#$?保存最近一次命令的退出状态码，0表示成功，非0表示失败状态
```



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
#name按名查找，iname表示忽略大小写
find / -name xxx
#mtime按修改时间查找，atime按访问时间查找，ctime按创建时间查找，+为查找3214天前修改的文件，-3214查找3214天前到现在期间修改的文件，没有后+—代表查找3214天前这天修改的文件
find / -mtime /+/-3214
#size按大小查找，c代表字节，k千字节，M兆字节，G吉字节，+代表查找比该它大的，-代表查找比它小的
find / -size /+/-3243c/k/M/G
#按文件所有者查找
find / -user xxx
#exec，对查找的文件执行某个命令
find / -name xxx -exec cat yyy {} \;
#perm, 查找符合权限的文件(这里-u不是-user)
find / -perm -u=s 2>/dev/null
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

##### 14、循环和条件

while

```bash
while condition; do command; done

while condiftion; do
	command
done
```

```bash
#while循环提取一个文件中的每行数据赋值给变量以供使用
while IFS= read -r line; do readlink -e /etc/xdg/$line ; done < dict.txt
#while循环不停轮询一个文件中是否拥有内容并直到该文件存在内容为止
false; while [ $? -ne 0 ]; do cat /free/* ; done 2>/dev/null

###################################################################################
#while多层嵌套
###-eq判断数字类型相等，=判断字符类型相等(加"")
while IFS= read -r i; do count=0; while IFS= read -r j; do if [ $i = $j ]; then count=1; fi; done <2.txt; if [ $count -eq 0 ]; then echo $i; fi; done <1.txt 2>/dev/null 
###这里<2.txt用在done结束的地方代表重定向输入到该done所属的循环内
###展开形式,本质上不展开换行就用';'，但是展开换行就不用';''，展开只需要在while do间，if then间，for in与do间加';'即可。
while IFS= read -r i; do
	count=0
	while IFS= read -r j; do
		if [ $i = $j ]; then
			count=1
		fi
	done < 2.txt
        if [ $count -eq 0 ]; then
        	echo $i
        fi
done < 1.txt
2>/dev/null 
###################################################################################
```

for

```bash
#for循环生成一个特定的密码本，其中in后的内容主要有以下几种
#{a..z}是循环表达式，代表a到z的所有字母的遍历赋值给遍历，还可以嵌套循环表达式，例如{a..z}{a..z}。也可以用{00..99}生成所有的两位数，个位数默认给出前导0
#$(cat file.txt)读取file.txt每行文件赋值给变量 
#枚举{a,b,kk,37}
for var in condition; do command; done

for vat in condition; do
	command
done
```

if

```bash
#[的前后和]的前和判断符号的前后都必须有空格
###字符类型判断相等用=，数字型判断用-eq
if [ $a = $b ]; then command_1; elif [ $a = $c ]; then command_2; else command_3; fi

if [ $a = $b ]; then
	command_1
elif [ $a = $c ]; then
	command_2
else
 	command_3
fi
```



##### 15、ssh

本地端口转发：将本地某个端口映射到远程某个端口

```bash
#创建一个SSH隧道，将本地端口转发到远程主机上的指定端口
#该命令执行结果是：利用ssh连接到远程主机的5000端口，并将本地的9001端口收到的数据转发到远程主机的本地地址上的80端口
#lola@venus.hackmyvm.eu是远程主机，127.0.0.1:80是要转发到远程主机的某个地址的某个端口（这里是远程主机自己的本地地址）,127.0.0.1:9001是本地主机的哪个地址的哪个端口（这里是本地主机的本地地址，一般可以省略，默认为本地地址）
ssh -L 127.0.0.1:9001:127.0.0.1:80 lola@venus.hackmyvm.eu -p 5000
```

动态端口转发：本地某个端口的数据通过ssh端口发送到远程

```bash
#本质上在本地某端口创建一个socks代理，将本地要发送到该端口的所有流量走该代理

```



远程端口转发：将远程端口映射到本地某个端口

##### 16、mysql

```bash
#显示所有数据库
show databases;
#显示当前数据库所有表
show tables;
#选择数据库
use xxx_base;
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
insert into xxx_table (row_1, row2, row3) values('', , '')
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

##### 16、alias

别名

```bash
#显示所有别名
alias
```

##### 17、strings

查看某二进制文件中所有可以提取和打印的字符串

```bash
strings file
```

##### 18、mount

挂载文件或者介质（包括iso）或者外部硬盘等

```bash
#创建挂载点,一般放在/mnt下，随意设置
mkdir -p /mnt/iso
#挂载普通文件，-o loop将文件作为块设备挂载
mount -o loop path/to/your.iso /mnt/iso
#挂载外部硬盘，找到硬盘的块设备路径/dev/sdb,/mnt/usb现创建
mount /dev/sdb /mnt/usb

#查看磁盘分区情况和块设备路径
sudo fdisk -l
#或者
lsblk
```

##### 19、id

查看当前用户所属uid和gid

```bash
#这里paula的用户id(uid)为1044，它的组id(gid)为1044
#所以他一定在组1044中，同时这里它也还在kala这个组id为1053的组中
paula@venus:~$ id
uid=1044(paula) gid=1044(paula) groups=1044(paula),1053(kala)
```

##### 20、doas

类似于sudo

```bash
#类似su user_1的命令
doas -u user_1 bash
```

##### 21、vim

```bash
#dw删除一个单词
#.重复上一个动作
#gg第一行，G最后一行

#:!后可以输入外部命令以当前运行文件的用户身份执行命令
#vi中是直接!后输入，不摁:
```

##### 22、suid

被设置为suid位的文件，在任何用户执行它时均会以文件拥有者的权限运行它而无需sudo提权（除非文件执行过程中需要访问一些高权限的文件）

suid只对二进制可执行文件有效，脚本等都无效

设置为suid不代表一定可以无需密码执行文件，除上述原因外还有原因详见sudo处

```bash
find / -perm -u=s -type f 2>/dev/null
```

##### 23、用户组

每个用户在系统中都有一个主组，在创建文件时如果不特别指定就会将该文件的用户组指向该主组。

可以指/etc/group中可以查看所有的用户组信息，也可以通过groups命令和id命令查看本用户所在的用户组情况

```bash
#/etc/group格式
#arg_1为用户组名，arg_2为密码默认用x隐藏，arg_3为组id，arg_4为组用户的用户列表
adm:x:4:username_1,username_2
```

##### 24、ps

```bash
#查看所有进程,e显示所有进程，L显示线程，f以完整格式显示进程信息
ps -ef
#lsof查看所有进程
#查看pid_id进程原始执行文件路径
ll /proc/pid_id/exe
#查看执行pid_id进程的原始完整命令行命令
cat /proc/pid_id/cmdline
#查看pid_id进程的名称
cat /proc/pid_id/comm
```

##### 25、top

查看当前cpu使用情况

```bash
#在top下H可以查看所有线程信息，c可以切换COMMAND字段（在cmdline和comm之间变换）
top
```

26、lsof

查看系统中打开的所有文件，包括文件描述符、套接字、设备

```bash
lsof
```

##### 27、cron

用于查看所有的定时任务

```bash
#当前用户定时任务，l查看，e编辑，-u指定某用户定时任务进行操作(需sudo)
#实际文件位于/var/spool/cron/*
crontab -l

#查看系统级别的定时任务，它们存在于
#/etc/crontab、/etc/cron.d/、/etc/{hourly,daily,weekly,monthly}/中
#crontab是简易的定时任务，cron.d是复杂的模块化的定时任务，{}是固定每x执行一次的任定时任务

#anacron异步定时任务，是指期望运行时间处于非其运行级别下的任务在切换到其运行级别时马上执行以实现异步效果，/etc/anacrontab中指定了哪些cron中哪些脚本应为异步定时的，它是cron的补充。
#/var/spool/anacron/中指定了各个异步定时任务上次的执行时间，系统启动时通过检查上次执行时间和当前时间，再查看脚本定义的执行间隔时间即可明白是否执行该任务实现异步。
```

##### 28、netstat

查看当前主机的所有网络情况

```bash
#a：显示所有的网络连接，包括监听和非监听状态。
#n：以数字形式显示地址和端口（而不是解析成主机名和服务名）。
#u、t：显示UDP、TCP协议的连接。
#l：只显示监听状态的连接。
#p：显示使用这些连接的进程和 PID 信息（需要 root 权限）
netstat -altunp
```

##### 29、init.d和服务自启（现代主要使用systemd，但是还是叫做init.d）

管理系统初始化时要执行的任务和运行的程序

```bash
#分为6个运行级别，对应了6个/etc/rc*.d文件，这些文件里是系统该运行级别下初始化任务
#0:关机 
#1:单用户模式 
#2:多用户模式(无网络) 
#3:多用户模式(有网络) 
#4:用户自定义 
#5:多用户模式(有个网络和图形界面) 
#6:重启

#查看当前运行级别
runlevel
#系统默认的运行级别
vi /etc/inittab

#/etc/rc.d/init.d/下为进入或离开某个运行级别时的服务启动脚本（该文件夹会被符号链接到/etc/init.d/），每个脚本对应一个服务，规定了这个服务该在什么运行级别下或其它什么情况下运行。每个脚本会被符号链接到具体的运行级别文件夹下（即/etc/rc.d/rc0.d/、/etc/rc.d/rc1.d/以此类推）。

#/etc/rc.d/rc.local/下为所有其他初始化（初始化即进入或离开）脚本运行之后执行的脚本（符号链接到/etc/rc.local/），可以放自定义的脚本。

#/etc/rc.d/rc脚本是初始化的启动脚本（符号链接到/etc/rc），用于执行/etc/rc.d/rc*.d/下的脚本。
```

##### 30、服务自启

其实就是对于init初始化的一个补充

```bash
#服务自启动除了在初始化时直接在rc*.d/配置外，还有以下配置方式

#1、通过/etc/rc.d/rc.local，可以在初始化完成后自动执行命令，例如运行/etc/init.d/下的脚本，只需要将要执行的脚本和命令添加在该文件中即可。
echo "/path/to/exec start" >> /etc/rc.d/rc.local


#2、通过chkconfig控制服务自启动时，自启动脚本应位于/etc/init.d/下
#查看控制的自启服务
chkconfig --list
#打开或关闭服务自启，level指明自启的运行级别，默认使用2344运行级别
chkconfig (–-level 2345) httpd on/off
 

#3、使用ntsysv管理自启动(基于RED HAT)
```

