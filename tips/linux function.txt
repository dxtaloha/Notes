### 1、加用户

sudo adduser newuser //创建

sudo usermod -aG sudo newuser //root权限

sudo passwd newuser //设置密码

### 2、kernel

lsmod |grep 关键字 //查内核驱动模块

rmmod卸载，insmod加载，modprobe -r卸载，modprobe加载

lspci -nn //查pci设备

ethtool -i 网卡名 //查网卡信息

ifconfig 网卡名 down(up) //开关网卡

lscpu //查看cpu状态

tail -f syslog //位置位于/var/log,日志文件为syslog

xgrgs rm < install_manifest.txt //卸载make install的文件或make uninstall

strace -ff -o output.file 执行的程序(代码文件)路径 //output.file是输出的地方，跟踪程序的执行，包括系统调用