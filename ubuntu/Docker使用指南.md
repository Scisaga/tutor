## 基本概念
- 镜像
   - 操作系统分为内核和用户空间。对于 Linux 而言，内核启动后，会挂载 root 文件系统为其提供用户空间支持。而 Docker 镜像（Image），就相当于是一个 root 文件系统
- 容器
   - 容器是Linux为了解决虚拟机问题而出现的一种虚拟化技术，Linux 容器（Linux Containers，缩写为 LXC）。
   - 容器不是对操作系统的模拟，而是对系统进程的一种隔离。
   - 优点：启动快，体积小，资源占用少
- 仓库
   - 镜像托管之地
- 优点
   - **提供一次性的环境**。比如，本地测试他人的软件、持续集成的时候提供单元测试和构建的环境。
   - **提供弹性的云服务**。因为 Docker 容器可以随开随关，很适合动态扩容和缩容。
   - **组建微服务架构**。通过多个容器，一台机器可以跑多个服务，因此在本机就可以模拟出微服务架构。

## 安装


## 容器管理
```shell
# 查看容器
docker ps -a

# 容器/镜像资源占用
docker system df -v

# 进入容器
docker exec -it <container_name> /bin/bash

# 查看日志
docker logs <container_name>
# 终端实时输出
docker logs -f <container_name>
# 输出到文件
docker logs <container_name> > container_name.log

```