---
typora-copy-images-to: ..\photo

---

## 先决条件

1、Burpsuite需要配置java环境

2、如果要劫持网页流量需要先配置网页代理，然后配置浏览器证书，浏览器证书从127.0.0.1:port或者http://burp/访问下载。然后在网页中搜索关键字cert，将下载的证书导入并信任授权

![image-20240323160305748](../photo/image-20240323160305748-1712922088818.png)

***如果是远程截取网页信息，需要在option里配置本机的ip:port，远程被拦截主机网页需要配置代理为这个拦截主机在公网的ip:port。***



## 激活

### windows

![image-20240325092401434](../photo/image-20240325092401434-1712922110389.png)

在解压后的文件夹下打开cmd输入该命令生成lincense

![image-20240325092510590](../photo/image-20240325092510590-1712922117490.png)

![498e9764c3214317afc775028c1bba97](../photo/498e9764c3214317afc775028c1bba97-1712922139054.png)

![image-20240325092535625](../photo/image-20240325092535625-1712922148623.png)

### Linux

1、首先需要安装专业版才能破解。

2、安装过程和windows是一样的，把BurpLoaderKeygen.jar和burpsuite_pro.jar放入/usr/bin中（放在同一个文件目录下），然后用java -jar xxx.jar运行Keygen。

3、然后在Keygen中run，打开burpsuite激活页面激活，激活完后先退出。

4、更改burpsuite命令行快捷启动版本。apt卸载原先的社区版，然后在/usr/local/bin（这是系统环境变量PATH的默认查找路径）下建立一个burpsuite.sh脚本，写入

```sh
#!/bin/bash
//新一行写入Keygen界面中run左边的loader command命令，把双引号都删掉即可。(因为需要其用密钥生成器引导启动)
```

将其改名后使其可执行

```sh
sudo mv burpsuite.sh burpsuite
sudo chmod +x burpsuite
```



## dashboard

爬虫，默认为爬proxy拦截的网页

## target

爬出的内容

## proxy

拦截

## intruder

爆破，可以将拦截的内容的特定字段进行爆破，需要字典

![image-20240324222003616](../photo/image-20240324222003616-1712922192547.png)

302表明了爆破成功（有的时候200也不一定代表成功），302代表重定向，重定向到了该ip服务器的某个文件下，下图的location即为重定向到的文件目录

![image-20240324222107950](../photo/image-20240324222107950-1712922201191.png)



## repeater

重放http请求，但有一个问题，如何在重放一个正确的请求以后获取后续的内容？