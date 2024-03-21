# 1、网络

### 配置ip

/etc/network/interfaces

### 配置DNS

/etc/resolve.conf

### 重启服务

service networking restart

service network-manager reastart

### 自动获取ip

dhclient

## 2、git代理

git config --global http,proxy 'socks5://127.0.0.1:1080'

git config --global https.proxy 'socks5://127.0.0.1:1080'

把ip改成网关地址（虚拟机上，ip为vmnet8的地址）

## 3、ssh

vim /etc/ssh/sshd_config   //开启密码登录，允许root用户登录（PasswordAuthentication\PermitRoot)

service ssh start //开启ssh，restart是重启