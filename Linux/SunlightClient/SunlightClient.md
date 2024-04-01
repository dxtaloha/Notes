## 安装图形版

1、官网安装下载.deb文件

2、查看自己是x11还是wayland图形界面，一般都是wayland，不是的话改一下

3、sudo dpkg -i xxx.deb执行安装

如果安装失败：

```bash
vim /etc/apt/source.list


deb http://cz.archive.ubuntu.com/ubuntu bionic main universe  //添加该源,如果报错添加公钥即可
sudo apt-key adv --keyserver keyserver.ubuntu.com --recv-keys 3B4FE6ACC0B21F32//添加公钥（不一定要执行）


sudo apt-get update
sudo apt-get install -f //安装相应依赖
sudo dpkg -i xxx.deb  //重新安装
```

## 打开向日葵

普通用户身份执行：

/usr/local/sunlogin/bin/sunloginclient