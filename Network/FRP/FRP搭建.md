## 简介

只要有一个公网ip，frp就可以用这个公网ip进行穿透，本质上frp就是进行了一个端口映射。

下载地址：

https://github.com/fatedier/frp/releases/download/v0.51.3/frp_0.51.3_linux_amd64.tar.gz  //这个是amd的

https://github.com/fatedier/frp/releases/download/v0.51.3/frp_0.51.3_linux_arm64.tar.gz  //这个是arm的

frp服务端和客户端都在同一个文件夹下，服务端是frps，客户端是frpc。frps.ini和frpc.ini分别是他们的配置文件。

## 服务端配置

```bash
sudo chown -R root:root frp_0.51.0_linux_amd64 //frp运行需要root权限
cd frp_0.51.0_linux_amd64

sudo vim ./frps.ini
###配置文件内容，注意配置文件中不能出现#之类的注释符，行尾不能有空格，否则可能无法运行
[common]
bind_port = 7001  //作为frp上服务端和客户端保持连接的服务端端口，云服务器需要去云上打开7001端口
token=123456  //认证令牌，客户端同样配置即可
###

###用systemd将frps加入，使其可以用systemd管理,下为frps.service的模板内容
[Unit]
Description = frp server  //别名，没啥用
After = network.target syslog.target
Wants = network.target

[Service]
Type = simple
ExecStart = /home/dxt/frp/frps -c /home/dxt/frp/frps.ini
Restart=on-failure

[Install]
WantedBy = multi-user.target
###然后启动、配置为开机自启

```



## 客户端配置

```bash
sudo chown -R root:root frp_0.51.0_linux_amd64 //frp运行需要root权限
cd frp_0.51.0_linux_amd64

sudo vim ./frpc.ini
###配置文件内容，注意配置文件中不能出现#之类的注释符，行尾不能有空格，否则可能无法运行
[common]
server_addr = ip  //服务端公网ip
server_port = 7001  //服务端用来和客户端保持连接的服务端端口号
token=123456  //认证令牌，客户端同样配置即可

[ssh203]           //这个名称必须是独一无二的
type = tcp    //需要监听的端口的数据类型，需要和远程用户要访问内网的服务所用的协议一样，还可以是http、udp等
local_ip = 127.0.0.1  //或者是192.168.6.x，是本机器的ip
local_port = 22  
remote_port = 7888  //设置映射端口，加上local_port的意思就是把公网ip的22端口映射给本机器的7888端口
###

###用systemd将frpc加入，使其可以用systemd管理,下为frpc.service的模板内容
[Unit]
Description = frp client
After = network.target syslog.target
Wants = network.target

[Service]
Type = simple
ExecStart = /home/dxt/frp/frpc -c /home/dxt/frp/frpc.ini
Restart=on-failure

[Install]
WantedBy = multi-user.target
###然后启动、配置为开机自启

```

