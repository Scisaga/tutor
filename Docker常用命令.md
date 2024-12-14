## Docker介绍
Docker 是由 Google 公司推出的 Go 语言开发实现的一种基于 Linux 内核的虚拟化技术，它依赖于内核中的 cgroup、namespace 以及 AUFS 类的 UnionFS 等技术，通过对进程进行封装和隔离，实现了操作系统层面的虚拟化。

在功能层面，Docker 能够自动化处理重复性任务，例如搭建和配置开发环境，从而显著提升开发效率。用户可以方便地创建和管理容器，将应用程序封装至容器内。容器支持版本管理、复制、分享和修改，使用方式类似于代码管理，极大地优化了开发者的工作流。

Docker 提供的镜像包含除内核外完整的运行时环境，这确保了应用运行环境的一致性。基于此，Docker 实现了秒级甚至毫秒级的启动时间，显著缩短了开发、测试和部署周期。同时，其优秀的弹性伸缩能力能够快速应对集中爆发的服务器使用压力，灵活实现资源扩展。

此外，Docker 的跨平台迁移能力极强，可以轻松将应用从一个平台迁移至另一个平台，而无需担心运行环境变化导致的不兼容问题。通过定制应用镜像，Docker 还能够轻松实现持续集成、持续交付与部署，成为现代软件开发的重要工具。

> **容器化技术** 是将软件打包制作成标准化单元的技术，可用于开发、交付和部署，容器化技术具有以下特征：
> 1. 容器镜像是轻量的、可执行的独立软件包，包含软件运行所需的所有内容：代码、运行时环境、系统工具、系统库和设置。
> 2. 容器化软件适用于基于Linux和Windows的应用，在任何环境中都能够始终如一地运行。
> 3. 容器化赋予了软件独立性，使其免受外在环境差异（例如，开发和预演环境的差异）的影响，从而有助于减少团队间在相同基础设施上运行不同软件时的冲突。

### 基本概念
#### 镜像 Image
- Docker镜像是一个特殊的文件系统，除了提供容器运行时所需的程序、库、资源、配置等文件外，还包含了一些为运行时准备的一些配置参数（如匿名卷、环境变量、用户等）。
- 镜像不包含任何动态数据，其内容在构建之后也不会被改变。
- 镜像实际是由多层文件系统联合组成。分层存储的特征使得镜像的复用、定制变的更为容易。可以用之前构建好的镜像作为基础层，然后进一步添加新的层，以定制自己所需的内容，构建新的镜像。
- 可以使用Dockerfile格式的脚本语言进行镜像定义与构建。

#### 容器 Container
- 镜像和容器的关系，就像是面向对象程序设计中的类和实例一样，镜像是静态的定义，容器是镜像运行时的实体。容器可以被创建、启动、停止、删除、暂停等。
- 容器的实质是进程，但与直接在宿主执行的进程不同，容器进程运行于属于自己的独立的命名空间。前面讲过镜像使用的是分层存储，容器也是如此。
- 容器存储层的生存周期和容器一样，容器消亡时，容器存储层也随之消亡。因此，任何保存于容器存储层的信息都会随容器删除而丢失。
- 容器不应该向其存储层内写入任何数据 ，容器存储层要保持无状态化。所有的文件写入操作，都应该使用数据卷、或者绑定宿主目录，在这些位置的读写会跳过容器存储层，直接对宿主(或网络存储)发生读写，其性能和稳定性更高。数据卷的生存周期独立于容器，容器消亡，数据卷不会消亡。因此， 使用数据卷后，容器可以随意删除、重新运行 ，数据却不会丢失。

#### 仓库 Registry
Docker Registry提供集中的存储、分发镜像的服务： 
- 一个 Docker Registry中可以包含多个仓库； 
- 每个仓库可以包含多个标签（Tag）； 
- 每个标签对应一个镜像；
- 可以通过<仓库名>:<标签>的格式来指定具体是这个软件哪个版本的镜像。如果不给出标签，将以 latest 作为默认标签。

Docker Registry公开服务是开放给用户使用、允许用户管理镜像的 Registry 服务。一般这类公开服务允许用户免费上传、下载公开的镜像，并可能提供收费服务供用户管理私有镜像。
最常使用的 Registry 公开服务是官方的Docker Hub，这也是默认的 Registry，并拥有大量的高质量的官方镜像。

用户还可以在本地搭建私有 Docker Registry 。Docker 官方提供了 Docker Registry 镜像，可以直接使用做为私有 Registry 服务。开源的 Docker Registry 镜像只提供了 Docker Registry API 的服务端实现，足以支持 docker 命令，不影响使用。但不包含图形界面、镜像维护、用户管理、访问控制等高级功能。

### docker-compose

Docker Compose 是一个用于定义和运行多容器 Docker 应用的工具。通过一个简单的 YAML 文件（通常命名为 docker-compose.yml），你可以配置应用的服务，并使用单个命令来启动或停止它们。

- 多容器编排
   - Docker Compose 允许在一个文件中定义多个服务（容器），并通过一个命令同时管理它们。
- 服务依赖管理 
   - 可以设置服务之间的依赖关系，确保它们以正确的顺序启动。例如，数据库服务应在应用服务启动之前运行。
- 跨环境一致性
   - 开发者可以在本地运行 Compose 来模拟生产环境，从而保证一致的行为。
- 网络支持
   - 默认情况下，Compose 会为定义的服务创建一个独立的网络，方便服务之间通信。

## 安装
```shell
curl -fsSL "https://get.docker.com" | bash -s docker --mirror Aliyun && \
systemctl enable docker && \
systemctl start docker

curl -L "https://github.com/docker/compose/releases/download/v2.29.7/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
chmod +x /usr/local/bin/docker-compose
```

## 常用命令
```shell
# 设定容器自启动
docker update --restart=always <CONTAINER ID>

# 查看容器IP
docker inspect -f '{{range.NetworkSettings.Networks}}{{.IPAddress}}{{end}}' ${container_name_or_id}

# 查看容器内进程
docker top ${container_name}

# 查询所有的容器，过滤出Exited状态的容器，列出容器ID，并删除这些容器
docker rm `docker ps -a|grep Exited|awk '{print $1}'`

# 查看容器磁盘使用
docker ps -s

# 查看每个image、container大小
docker system df -v

# 删除不再使用的挂载卷
docker volume prune

# ！删除所有容器
docker rm -f $(docker ps -a -q)

# ！删除所有卷
docker volume rm $(docker volume ls -q)
```
docker-compose
```shell
# 重新拉取镜像
docker pull ${镜像名}

# 启动容器
docker-compose -f ${yaml_file} up -d

# 销毁容器
docker-compose -f ${yaml_file} down

# 停止并销毁容器，同时删除容器所使用的虚拟磁盘卷空间
docker-compose down --volumes
```

## 配置

### `/etc/docker/daemon.json`
推荐配置
```json
{
    "data-root": "/data/volumes", # 配置容器所用 volumes 文件夹
    "log-driver": "json-file",    # 日志配置
    "log-opts": {
        "max-file": "3",
        "max-size": "100m"
    }
}
```

### 代理配置

#### dockerd代理
在执行 `docker pull` 时，是由守护进程 `dockerd` 来执行。 因此，代理需要配在 `dockerd` 的环境中。修改`/etc/docker/daemon.json`后重启docker服务
```shell
{
  "proxies": {
    "http-proxy": "http://proxy.example.com:3128",
    "https-proxy": "https://proxy.example.com:3129",
    "no-proxy": "*.test.example.com,.example.org,127.0.0.0/8"
  }
}
```

#### Container代理
在容器运行阶段，如果需要代理上网，则需要配置`~/.docker/config.json`。以下配置，只在Docker 17.07及以上版本生效。
```json
{
    "proxies": {
        "default": {
            "httpProxy": "http://proxy.example.com:8080",
            "httpsProxy": "http://proxy.example.com:8080",
            "noProxy": "localhost,127.0.0.1,.example.com"
        }
    }
}
```
这个是用户级的配置，除了proxies，docker login等相关信息也会在其中。 而且还可以配置信息展示的格式、插件参数等。

此外，容器的网络代理，也可以直接在其运行时通过-e注入http_proxy等环境变量。这两种方法分别适合不同场景。
- config.json非常方便，默认在所有配置修改后启动的容器生效，适合个人开发环境。
- 在CI/CD的自动构建环境、或者实际上线运行的环境中，这种方法就不太合适，用-e注入这种显式配置会更好，减轻对构建、部署环境的依赖。
- 当然，在这些环境中，最好用良好的设计避免配置代理上网。

#### docker build代理
虽然docker build的本质，也是启动一个容器，但是环境会略有不同，用户级配置无效。在构建时，需要注入http_proxy等参数。
```shell
docker build . \
    --build-arg "HTTP_PROXY=http://proxy.example.com:8080/" \
    --build-arg "HTTPS_PROXY=http://proxy.example.com:8080/" \
    --build-arg "NO_PROXY=localhost,127.0.0.1,.example.com" \
    -t your/image:tag
```

### docker-compose

## 网络

## 安全性
### docker信任自签发证书
1. 将自签发证书复制为 /etc/docker/certs.d/${domain}/ca.crt
   - domain：域名
2. `systemctl daemon-reload && systemctl restart docker`

## 宿主机
### 容器内访问宿主机
- `host.docker.internal`
- `gateway.docker.internal`

### 在容器中执行宿主机Docker命令
需要将宿主机的docker命令管道映射到需要执行docker命令的容器中，docker-compose配置示例如下
```yaml
volumes:
    - /var/run/docker.sock:/var/run/docker.sock
```
参考：
- [Running Docker in Docker on Windows (Linux containers)](https://tomgregory.com/running-docker-in-docker-on-windows/)
- [How To Run Docker in Docker Container [3 Easy Methods]](https://devopscube.com/run-docker-in-docker/)

## 日志
查看容器日志文件路径：
```sh
docker inspect --format='{{.LogPath}}' ${container_name}
```
清空容器日志：
```sh
$ sudo sh -c 'echo "" > $(docker inspect --format="{{.LogPath}}" ${container_name})'
# 或
truncate -s 0 $(docker inspect --format='{{.LogPath}}' ${container_name}) 
```
检视容器日志：
```sh
# 最新100条
docker logs ${container_name} --tail 100
# 最近1小时
docker logs ${container_name} --since 1h
# 最近一小时中的最新100条
docker logs ${container_name} --tail 100 --since 1h
# 终端实时输出
docker logs -f ${container_name}
# 输出到文件
docker logs ${container_name} > container_name.log
```

## Dockerfile
Dockerfile 是用于定义如何构建 Docker 镜像的文件，通过一系列指令描述构建步骤。

常用指令：
- FROM
   - 指定基础镜像，所有 Dockerfile 必须以 `FROM` 开头。
   - `FROM ubuntu:20.04`
- RUN
   - 执行命令，用于安装软件或执行配置操作。
   - `RUN apt-get update && apt-get install -y curl`
- CMD
   - 定义容器启动时的默认命令
   - CMD ["/bin/sh", "/opt/entrypoint.sh"]
- ENTRYPOINT
   - 设置容器的固定入口命令
   ```dockerfile
   ENTRYPOINT ["python3"]
   CMD ["app.py"]
   ```
- COPY
   - 将本地文件复制到镜像内。
   - `COPY app.py /app/`
- ADD
   - 类似于 COPY，但支持解压 .tar 文件或从 URL 下载。
- ENV
   - 设置环境变量
   - `ENV APP_ENV production`
- WORKDIR
   - 设置工作目录
   - `WORKDIR /app`
- EXPOSE
   - 声明容器对外开放的端口
   - `EXPOSE 8080`
- VOLUME
   - 定义匿名挂载点，用于持久化数据。
   - `VOLUME ["/data"]`
- HEALTHCHECK
   - 设置容器的健康检查
   - 执行指令退出码为0代表健康状态
   - `HEALTHCHECK --interval=10s --timeout=3s --retries=3 CMD curl -f http://localhost:80 || exit 1`

Python应用Dockerfile示例
```dockerfile
# 使用基础镜像
FROM python:3.9-slim

# 设置工作目录
WORKDIR /app

# 复制应用代码
COPY . /app

# 安装依赖
RUN pip install -r requirements.txt

# 暴露端口
EXPOSE 5000

# 设置健康检查
HEALTHCHECK --interval=10s CMD curl -f http://localhost:5000 || exit 1

# 启动命令
CMD ["python", "app.py"]
```

构建镜像
```shell
docker build -t my-python-app .
```

运行镜像
```shell
docker run -p 5000:5000 my-python-app
```

### 常见问题
#### 输出构建详细信息 + 不使用缓存
```shell
docker build --progress=plain --no-cache ...
```
#### 在 transferring context 这一步卡死
考虑是否有进程正在使用构建镜像所需的文件，譬如IDEA中还有正在执行的进程
