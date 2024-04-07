## 基本命令

![image-20240407201435197](Command.assets/image-20240407201435197.png)

## meterpreter

用于攻击后进行交互和后渗透等。他是基于内存dll注入实现的阿，能够创建一个新进程并调用注入的dll来让目标系统运行注入的dll文件，攻击者与靶机之间的meterpreter通信时通过stager套接字实现的。

![image-20240407204223319](Command.assets/image-20240407204223319.png)

## msfvenom

用于生成木马

**用msfvenom -l payloads查看当前msfvenom的可用payload，不同版本给定的-p的对象会不一样。**pyaload分为大马和小马，大马是stageless payload，小马是staged payload。staged payload类型为linux/x86/meterpreter/reverse_tcp，stageless payload类型为linux/x86/meterpreter_reverse_tcp。

特别的，不同的目标文件-f是不一样的，.php的-f是raw，.jsp也是raw，这个要现查一下

payload分为

![image-20240407204347397](Command.assets/image-20240407204347397.png)

示例

![image-20240407204415280](Command.assets/image-20240407204415280.png)

## 监听器（exploit/multi/handler)

用于监听端口，远程木马一旦执行即可和监听端口建立连接，从而可以使用meterpreter。

**特别的是，要设置payload，payload为msfvenom设置的payload类型。**