---
title: Docker常用命令
date: 2021-01-22 10:08:30
tags: [Docker]
---

# 1.docker  run     [可选参数]    镜像     [指令]

> 创建一个新的容器并运行一个命令

```shell
#可选参数

-i 以交互模式运行容器，通常与 -t 同时使用
-t 为容器重新分配一个伪输入终端，通常与 -i 同时使用
-d: 后台运行容器，并返回容器ID
-p: 指定端口映射，格式为：主机(宿主)端口:容器端口
--name="nginx-lb": 为容器指定一个名称



```



```shell

# e.g.
 docker run -itd -p5258:5258 shihd/gbase8a:1.0
```





# 2.docker  pull    镜像

```shell
docker pull shihd/gbase8a:1.0
```



# 3.docker save 镜像 > 包名.tar.gz

> 保存镜像到离线包

```shell
docker save d8f4c547d6ae > act-gbase8s.tgz
```



# 4.docker load 

> 导出打包好的镜像离线包，用于后续可离线安装

```shell
docker load -i gbase8a-1.0.tar
```



# 5.docker tag 镜像 tag名称

```shell
docker tag d8f4c547d6ae act-gbase8s:8.7.2
```

