> 这位更是不用说，神中神

docker 简单来说，就是将某套应用程序依赖的环境，文件封装独立的运行环境。每个**运行环境**就是一个容器， 镜像可以类比于应用程序的安装包。镜像是运行在容器上的。

镜像可以上传到镜像仓库，给其他人使用 - 官方镜像仓库： docker hub

docker 与 虚拟机的最大区别：
+ docker 是共用一套系统内核
+ 虚拟机 都包含一个操作系统的完整内核

docker 运行模式：
![450](assets/docker/file-20251207164328548.png)

虚拟机 运行模式：
![450](assets/docker/file-20251207164527433.png)


## 下载镜像
`docker pull docker.io/library/nginx:latest`  
+ docker.io - registry 仓库地址 
	+ docker.io 是官方维护的地址
+ library - 命名空间： 上传镜像的作者
	+ library 是官方
+ nginx - 镜像名
+ latest - 版本号： 默认最新

`docker pull --platform=xxx nginx:latest` 选择特定CPU架构的镜像

--- 
国内需要换一下镜像地址
```js
"registry-mirrors": [
    "https://docker.1ms.run",
    "https://docker.xuanyuan.me",
    "https://docker-0.unsee.tech",
    "https://docker-cf.registry.cyou",
    "https://docker.1panel.live"
  ]
```

## 查看所有镜像
`docker images` 

## 删除镜像
`docker rmi nginx / id `

## 运行镜像
`docker run nginx / id`

后台运行：
`docker run -d nginx`

端口映射： 默认情况下， 宿主机与容器是隔离的， 不能直接访问 容器的网络
`docker run -p 80:80 nginx` 
80:80 先外后内， 将宿主机的 80 端口映射为容器的 80 端口， 这样， 用户访问宿主机的 80 端看，其实访问的是容器的 80 端口

## 查看正在运行的容器
`docker ps`