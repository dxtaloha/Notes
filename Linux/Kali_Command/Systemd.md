## 简介

用systemd管理服务的系统均可以设置开机自启，并用systemd启动。

在/etc/systemd/system/、/lib/systemd/system、/usr/bin/systemd/system下有所有的可以用systemctl启动的程序，并可以为他们配置开机自启。

不同的是/etc/中为最高级，优先级最高，可以覆盖/lib和/usr。

而/usr/往往是符号链接到了/lib，他俩此时本质上是一个，但是如果不是符号链接，那么/lib/中的要比/usr/中的高一级。

## 用法

1、将可执行程序添加到systemd从而使用systemctl启动

在/etc/systemd/system/下创建一个文件

```bash
###在/etc/systemd/system/下创建一个文件
sudo vim program.service

###在此文件下写入以下内容
[Unit]
Description=Your Program 1

[Service]
ExecStart=/path/to/program1  //可执行程序路径
Restart=always

[Install]
WantedBy=multi-user.target

###设置开机自启以及启动该服务
sudo systemctl enable program.service
sudo systemctl start program.service


##注意，该目标程序需要可执行
```



## 与环境变量的区别

systemd启动某个服务后是在后台启动，而且可以配置为开机自启。

环境变量只适用于临时启动，不是在后台运行。