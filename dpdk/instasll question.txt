#### 1、指定版本

pip3 apt install meson ninja

export PATH=~/.local/bin=$PATH 指定的版本号（仅在当前终端临时可用）

否则可能dpdk需要的版本不是需求的版本

### 2、重复定义pthread

升级glibc库

下为ubuntu18升级glibc

deb http://security.debian.org-security buster/updates main

在/etc/apt/sources.list添加一行

apt update -> sudo apt-key adv --keyserver keysever.ubuntu.com --recv-keys 112695AE562B32A 54404762BBB6E853(箭头指输了以后的报错的解决办法)

apt install libc6

### 3、以太网配置参数错误

①txq、rxq值超过max_rx_queues和max_tx_queues，在dpdk/drivers/net 对应网卡设备中设置（base文件夹下）

