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
Restart=on-failure  //后边有详解

[Install]
WantedBy=multi-user.target

###更改完成后重载systemd配置
sudo systemctl daemon-reload

###设置开机自启以及启动该服务
sudo systemctl enable program.service
sudo systemctl start program.service


##注意，该目标程序需要可执行
##执行x.service和x是一样的
```



## 与环境变量的区别

systemd启动某个服务后是在后台启动，而且可以配置为开机自启。

环境变量只适用于临时启动，不是在后台运行。



## systemd restart字段

- no：默认值，表示不自动重启服务。
- always：无论服务退出状态如何，总是尝试重启服务。
- on-success：只有当服务正常退出（退出状态码为0）时才重启服务。
- on-failure：只有当服务异常退出（包括因为未捕获的异常、信号终止等）时才重启服务。
- on-abnormal：只有当服务因为信号或超时而终止时才重启服务。
- on-abort：只有当服务因为未捕获的信号而终止时才重启服务。
- on-watchdog：只有当服务因为看门狗超时而终止时才重启服务。