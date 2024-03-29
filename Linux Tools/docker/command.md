## 基本操作

docker ps //显示当前容器运行情况，加-a会显示已关掉的容器

docker image ls //显示当前docker的镜像情况

docker pull 参数（例ubuntu） //从官方仓库拉取镜像

docker run -itd 镜像 --name 名字 bash //i:interface,t:tgerminal,d:detached,bash实为/bin/bash

docker exec -it 容器id bash //以-d后台创建容器时，需用exec进入

docker rm -f 容器id //删除容器

docker stop/start 容器id // 停止，启动一个容器

docker export 容器id > 路径/名 .tar //导出容器，后缀为tar

cat 路径/名 .tar | docker import - 镜像名/TAG //将快照导入为镜像，TAG为版本或标识，自定义

docker import URL/目录 //从URL或镜像导入容器快照

## 配置加速源

```bash
vim /etc/docker/daemon.json   //没有就新建一个
{
 "registry-mirrors": ["http://hub-mirror.c.163.co"]
}   //添加进去
```

## 根据.yml文件创建特定配置容器

```bash
sudo apt install docker-compose
sudo docker-compose up -d   //在.yml文件目录下执行
```

