## 简介

shadowsocks可以只安装服务端，另一端就可以用clash等工具通过在其配置文件中写入服务端的ip和端口和算法和密码构建vpn隧道，当然，也可以用shadowsocks的客户端实现。

**如果想通过vpn实现"真正"接入某个内网一样，即通过vpn接入该内网的远程主机想要获得一个该内网的ip，那么就需要开启TAP模式，即桥接模式，该模式本质上是在vpn服务端上创建了一个网桥，分配给利用vpn客户端接入该内网的每个远程主机设备。**

*但是这种情况必须要安装对应的vpn客户端才能实现，用clash配置大概里是无法实现的。*

在本处由于我用的是arm架构，包管理器中只有shadowsocks-libev，因此此处教程shadowsocks-libev，shadowsocks大差不差，可以网上查。

## 服务端搭建

### 结构：

shadowsocks-libev的配置文件在/etc/shadowsocks-libev中，config.json这个默认的配置文件是ss-server的（但是好像不应该又server port）。

shadowsocks-libev的可执行文件在/usr/bin/下，分为ss-server、ss-local、ss-redir、ss-tunnle四个。

shadowsocks-libev有一个默认的systemd.service文件，在/lib/systemd/system/下

一共有ss-server、ss-local、ss-redir、ss-tunnle四个.service，同样对应四个shadowsocks-libev的四个可执行文件，其中ss-server就是服务端，ss-local就是客户端。

加上一个默认.service文件，共有五个.service文件，其中四个的结构是shadowsocks-libev-xxx@.service，其中默认.service文件为shadowsocks-libev.service。默认.service默认执行默认配置文件config.json的ss-server服务，不用管；其次这个@后的意思为把@和.service之间的内容作为ss的.json配置文件名前缀输入，从而利用该配置文件启动一个ss服务(不用输入.json,只输入前缀)

**可以用sudo systemctl enable /lib/systemd/system/shadowsocks-libev-xxx@aaa.service配置用aaa这个配置文件开机自启ss-server服务。**

### 搭建：

```bash
sudo apt-get update
sudo apt-get shadowsocks-libev

sudo vim /etc/shadowsocks-libev/ss-server_7888.json
###服务端配置文件内容
{
    "server":"0.0.0.0",  //服务端监听哪里来的数据包
    "mode":"tcp_and_udp", //服务端监听来的数据包的类型
    "server_port":7888, //服务端监听哪个端口来的数据包
    "password":"IJWK4lqtgxzDN1dWE9Rfdg", //密码
    "timeout":300,
    "method":"chacha20-ietf-poly1305", //加密算法
    "fast_open":false //快速打开tcp
}
###

###将本配置文件用ss-server执行，并配置开机自启
##路径在/lib/systemd/system/下
sudo systemctl start shadowsocks-libev-server@ss-server_7888.service
sudo systemctl enable shadowsocks-libev-server@ss-server_7888.service
```

