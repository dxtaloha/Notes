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
#这里指定了dxt运行哪些需要超级权限的命令时不需要输入dxt的密码即可运行，没有NOPASSWD代表全需要密码
sudo -l

#例如这个xargs是sudo时不需要密码，但是不用sudo运行可能是无法运行的，因为没有权限，这里的NOPASSWD是用sudo提升权限运行它且不需要密码，而不是说不用sudo就可以直接运行它
```

6、查看某些命令在设置SUID位或者有sudo NOPASSWD等权限时如何提权

```bash
#几乎统计了绝大多数可执行命令在有漏洞的情况下提权的方法
https://gtfobins.github.io/
```

