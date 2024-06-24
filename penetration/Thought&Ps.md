1、当发现php页面的时候，一般都是？后面传参才有具体的页面回显，因此可以用ffuf爆破一下参数名和参数值试试

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

查看SUID文件

```bash
#SUID允许用户以文件拥有者的权限执行文件，而不是以调用者的权限。
#虽然是以拥有者的权限执行，但是不同的文件可能依旧有不同的机制要求调用者输入密码才能执行（比如sudo，执行sudo这个文件是以root执行的没要密码，但是用sudo执行它的对象文件时还是得要密码，因为它的对象文件需要执行就是需要root密码）
#SUID只为二进制可执行文件设置

#查找所有被设置为SUID的文件
find / -perm -u=s -type f 2>/dev/null
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

