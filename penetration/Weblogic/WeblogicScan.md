## 下载安装使用

https://github.com/rabbitmask/WeblogicScan下载

利用python3打开WeblogicScan.py，其中requirementst.txt为使用该python脚本的依赖，需要提前下载。

安装依赖：

```bash
pip3 install -r requirements.txt
pip3 install -r requirements.txt -i https://pypi.tuna.tsinghua.edu.cn/simple //如果下载有问题用清华源

```

target.txt为扫描地址和端口。

执行扫描：

```bash
python3 WeblogicScan.py -f target.txt
```

## 获取资产

shodan、fofa、钟馗之眼

```txt
app="BEA-WebLogic-Server"
```



google hacking

```txt
inurl:漏洞地址
intitle:weblogic等
```



