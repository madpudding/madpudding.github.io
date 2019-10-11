---
title: "docker删除镜像流程"
date: 2019-10-11T11:51:02+08:00
draft: false
---
### 想要删除docker里面的一个服务镜像，结果弄了一会儿没删掉，反思了一下,应该是这样的.
### 首先是先停掉容器，先用docker ps -a，查看容器id，然后docker stop掉容器的服务。
### 再删掉容器 docker rm 容器id，有些服务依赖不止一个容器，这个要注意。
### 最后，先用docker images查看镜像id，再 docker rmi 镜像id.