1、当发现php页面的时候，一般都是？后面传参才有具体的页面回显，因此可以用ffuf爆破一下参数名和参数值试试。或者换一种提交数据的方式，GET、PUT、POST、DELETE。或者可能需要特殊的用户代理，设置-A为特殊的用户代理。

http是可以自定义头部的关键字的，传入方式为"key:a",不是用"key=a"

http头的host字段指定了要访问的具体服务器，一个物理服务器（一个公网ip）是可以虚拟出多个虚拟服务器供用户访问不同网站使用的

2、拿到服务器权限后（www-data），该权限起始位置一般位于/var/www/html下，但是html下的文件一般没有写权限，因此要wget下载等要出去该位置。

3、文件有密码的话可以通过john和libreoffice2john尝试破解密码。

4、信息搜集的时候可以通过gobuster这种工具爆破文件路径然后尝试访问

5、在拿到初步的权限后，进一步提权或者拿到其它用户权限的思路

查看定时文件

```bash
#系统定时文件
cat /etc/crontab
ls -la /etc/cron.d/
#用户定时任务
crontab -l
#系统定期任务
ls -la /etc/cron.daily/
```

查看SUID文件和SGID文件

```bash
#SUID允许用户以文件拥有者的权限执行文件，而不是以调用者的权限。
#虽然是以拥有者的权限执行，但是不同的文件可能依旧有不同的机制要求调用者输入密码才能执行（比如sudo，执行sudo这个文件是以root执行的没要密码，但是用sudo执行它的对象文件时还是得要密码，因为它的对象文件需要执行就是需要root密码）
#SUID只为二进制可执行文件设置

#通过运行SUID文件和SGID文件，就会在执行程序的过程中获得文件拥有者或组用户的身份，继而可以通过他们身份执行一些命令以查看或者执行另外某些文件。

#查找所有被设置为SUID的文件和SGID的文件
find / -perm -u=s -type f 2>/dev/null
find / -perm -g=s -type f 2>/dev/null
#查找具有特定能力的文件
/usr/sbin/getcap -r / 2>/dev/null
```

查看是否还有其它未开放但处于监听状态的端口

```bash
ss -altp
```

查看谁有超级权限

```bash
#通过sudo -l查看当前用户在运行什么可执行文件时有超级权限且不需要输入密码，从而提权
#可能需要超级权限,得看有没有
#User dxt may run the following commands on arroutada:
#    (ALL : ALL) NOPASSWD ALL
#或   (ALL : ALL) ALL
#或   (ALL : ALL) NOPASSWD /usr/bin/xargs
#这里指定了dxt在使用sudo提升权限时可以提升为任何一个用户身份来运行xargs而不需要输入dxt的密码，没有NOPASSWD代表全需要密码
sudo -l

#或   (natalia) NOPASSWD /bin/bash
#这里指定了当前用户可以使用sudo提升权限为natalia以运行/bin/bash而不需要输入当前用户的密码，用-u指定要用sudo提升为哪个用户的权限
sudo -u natalia /bin/bash

#例如这个xargs是sudo时不需要密码，但是不用sudo运行可能是无法运行的，因为没有权限，这里的NOPASSWD是用sudo提升权限运行它且不需要密码，而不是说不用sudo就可以直接运行它
```

6、查看某些命令在设置SUID位或者有sudo NOPASSWD等权限时如何提权

```bash
#几乎统计了绝大多数可执行命令在有漏洞的情况下提权的方法
https://gtfobins.github.io/
```

7、有些特殊文件看起来很奇怪可能是进行编码后的某种文件，比如这里=结尾，猜测是base64编码，涌base64解码文件后用file查看文件格式，从而获取某些有用信息（这里其实是.jpg文件）

<img src="Thought&amp;Ps.assets/image-20240624152319253.png" alt="image-20240624152319253" style="zoom: 33%;" />

8、无法用ls访问当前目录下的文件及其信息不代表不能通过猜测文件名来直接访问到某个文件的内容（可能只禁用了用户ls的权限， 但并没有禁用查看某个文件的权限）

9、从外网访问某个网站以爆破文件路径时，可能有些服务器的配置会导致外网无权限访问而返回404进而导致gobuster之类的工具无法准确判断是否存在这样一个文件。但可能内网的某个用户是有这个权限访问该网页而不会返回404的，因此可以通过ssh将 HTTP 服务隧道传输到本地计算机的某个端口，然后用gobuster进行破解。

##### 进一步来说，外网用户不可以通过http端口访问到/var/www/html下的某个文件，不代表内网用户不可以通过http端口等访问到/var/www/html下的某个文件；同时内网用户可以通过http端口等访问/var/www/html下的某个文件但也并不代表其可以直接在命令行中用cat获取到这个文件的内容（用户没有读权限）。

10、通过id命令查看当前用户所在的用户组，然后用find查找该用户组的文件，则当前用户可能可以访问和执行这些组用户文件（ls -al查看权限方知），通过访问这些文件进行进一步操作。

11、看到一串编码，考虑是不是base64编码，还是hash

```bash
#base64典型为以=结尾
#MD5-based格式为 $1$salt$hash
#SHA-512-based格式为 $6$salt$hash
#SHA-1-based格式为 $sha1$salt$hash
#SHA-512-based格式为 $6$salt$hash
#Blowfish-based格式为 $2a$salt$hash
```

12、查看用户的dns配置文件

dns配置文件定义了域名和IP地址的映射、设置DNS记录、指定名称服务器等

位于/etc/bind下

13、在内网移动时，如果当前用户可以通过suid等执行会以其它人甚至超级用户的权限执行的文件，如果该猜测到该文件会执行输入的命令，那可以利用;/bin/bash来拿到该用户的shell（;后加其它命令例如cat xxx可能会被过滤空格，因此可能无法成功执行）

14、对于被怀疑的可执行文件，可以通过strings查看其可执行文件中的字符串判断其相关或控制或影响的文件等。

15、某用户以root身份运行某程序时，如果打开了某个只能以root用户身份运行的文件但是没有关闭(句柄，fd)，此时该用户可以去/proc/self/中查看这个文件句柄，也可以去这个由于未关闭打开的文件导致pid一直运行的的/proc/pid_value/fd中查找这个文件句柄，通过输出重定向查看这个fd中的内容。（虽然是root身份运行的程序，但是产生的位于/proc下该进程的信息都是属于该用户所有的，所以可以查看）

16、查看网页时最好打开f12，随时查看elements和network，避免一些隐藏的元素在网页中但是不被浏览器解析显示

17、容器逃逸

```bash
###1、检测是否在容器中
#主要检测方法
fdisk -l #如果空或者找不到命令就是在容器里
#
cat /proc/1/cgroup #如果有docker、lxc、k8s字眼的是容器，没有也不一定
#
ls -al / #根目录下有.dockerenv证明是容器（虚拟机里的容器不会有该文件）

###2、逃逸
```

18、iptables和iptables6

```bash
#开放0.0.0.0:22端口不代表可以访问，因为可能被防火墙例如iptables过滤了
#但是iptables只能过滤ipv4，不过滤ipv6，很有可能ipv6的22端口还处于开放状态，因此可以通过ping6或者
#nmap -6 -s -Pn fe80::/64查找所有ipv6。


#不仅如此，可能很多情况下靶机的用户都会忘记对ipv6配置防火墙规则，而只对ipv4配置了防火墙规则
```

19、

```bash
#双引号引起来的判断是按位判断，一个字符一个字符对比，通配符*是无效的
if [[ "$PASSWORD" == "$USER_PASS" ]] ; then
fi
#而不加双引号的判断不是按位判断的，是可以用通配符直接使其相等的
if [[ $PASSWORD == $USER_PASS ]] ; then
fi
```

20、脚本在执行命令时，执行的每个命令都会在进程中显示，也就是说利用进程监控可以看到每个命令的具体参数

```bash
#举例，该脚本中判断语句赋值语句部分等并不是执行了一个命令
#但是/usr/bin/echo等这种确实一个执行的命令，会被进程监控监控到
PASSWORD=$(/usr/bin/cat /root/pass)
read -ep "Password: " USER_PASS
if [[ $PASSWORD == $USER_PASS ]] ; then
  /usr/bin/echo "Authorized access"
  /usr/bin/sleep 1
  /usr/bin/zip -r -e -P "$PASSWORD" /opt/backup.zip /var/www/html
else
  /usr/bin/echo "Access denied"
  exit 1
fi

#利用watch配合pa持续监控命令的执行得到命令执行的参数，进而获得一些敏感信息例如上述脚本的$PASSWORD
watch -n 0.1 -d "ps aux |grep -ai /usr/bin/zip >> log.txt"
cat log.txt |grep /usr/bin/zip
```

21、

可执行文件就用strings何IDA查看逆向

非可执行文件，量小的就挨着查看，量大的就用grep -r key_word过滤关键字，一般过滤user、password、pass这种关键字

22、如果a软件可以集中管理b软件，那么可以尝试用b的漏洞在a中实现，比如hmv.adria中的scalar是管理git的工具，git可以被利用的漏洞scalar可以类似的利用

23、存在某些可执行文件的help或者man（可执行文件说明），在输入参数help查看时会由于内容过度导致采用分页器less进行查看，如果help的原文件是脚本等，边查看边解释运行，则此时可以通过直接在解释的过程中输入!/bin/bash来获得一个运行该文件用户身份的shell。（见hmv.adria)