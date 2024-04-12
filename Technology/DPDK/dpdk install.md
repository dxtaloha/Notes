### 1、安装malleanox驱动

1、下载.iso，挂载到mnt: mount iso /mnt/xxx

2、在/mnt/xxx下执行./mlxofedinstall --all

3、/etc/init.d/openibd restart

ps:若提示ib_core is in use,则sudo systemctl stop ibacm_socket，特别地，IB需选择eth.ib模式

(mlxconfig -d <mst device> query |grep LINK_TYPE)

(mlxconfig -d <mst      > set LINK_TYPE_P1/2=1/2/3),1为IB，2为eth，3为auto

<mst device>:安装mft，执行install.sh安装，然后mft start，mst status

mlnx _tune查看网卡情况，ibstat也是。

### 2、下载DPDK，下载依赖项，配置大页，挂载大页，绑定驱动

1、安装dpdk

sudo apt install build_essential

sudo apt install python-pip3

sudo apt install meson ninja //用export PATH=~/.local/bin:$PATH指定

sudo apt install pyelftools

sudo apt install libnuma -dev

sudo apt install pkg-config   //(pkgconf)

1、配置大页

vim /etc/default/grub:加一行，GRUB_CMDLINE_LINUX_DEFAULT="default_hugepagesz=1G hugepagesz=1G hugepages=8"  //如果想调整回默认的2MB，把这一行的内容设为空即可。

然后重启电脑（sudo update_grub可能也需要执行）

ps：查配置，grep Huge /proc/meminfo

2、挂载大页（永久挂载）

mkdir /mnt/huge

mount -t hugetlbfs pagesize=1GB /mnt/huge(不一定需要)

vim /etc/fstab:加一行，none /mnt/huge hugetlbfs pagesize=1G,size=8G 0 0

重启电脑

ps：查配置，cat /proc/mounts |grep hugetlbfs(出来的东西应该是.... 0 0.)