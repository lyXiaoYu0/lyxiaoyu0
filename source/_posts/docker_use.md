---
title: Docker_CentOS 安装使用指南
date: 2024-11-01 10:38:33
layout: post
categories: [Docker_CentOS 安装使用指南]
tags: [Docker]
comments: true
cover: /img/docker_logo.png
description: Docker_CentOS 安装使用指南
---

> 配置好了，可以装个宝塔玩哎，宝塔上面换镜像源容易点（主要是镜像源频繁的封）

# 1.安装Docker

Docker 分为 CE 和 EE 两大版本。CE 即社区版（免费，支持周期 7 个月），EE 即企业版，强调安全，付费使用，支持周期 24 个月。

Docker CE 分为 `stable` `test` 和 `nightly` 三个更新频道。

官方网站上有各种环境下的 [安装指南](https://docs.docker.com/install/)，这里主要介绍 Docker CE 在 CentOS上的安装。

# 2.CentOS安装Docker

Docker CE 支持 64 位版本 CentOS 7，并且要求内核版本不低于 3.10， CentOS 7 满足最低内核的要求，所以我们在CentOS 7安装Docker。



## 2.1.卸载（可选）

如果之前安装过旧版本的Docker，可以使用下面命令卸载：

```
yum remove docker \
                  docker-client \
                  docker-client-latest \
                  docker-common \
                  docker-latest \
                  docker-latest-logrotate \
                  docker-logrotate \
                  docker-selinux \
                  docker-engine-selinux \
                  docker-engine \
                  docker-ce
```



## 2.2.安装docker

首先需要大家虚拟机联网，安装yum工具

```sh
yum install -y yum-utils \
           device-mapper-persistent-data \
           lvm2 --skip-broken
```



然后更新本地镜像源：

```shell
# 设置docker镜像源
yum-config-manager \
    --add-repo \
    https://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
    
sed -i 's/download.docker.com/mirrors.aliyun.com\/docker-ce/g' /etc/yum.repos.d/docker-ce.repo

yum makecache fast
```





然后输入命令：

```shell
yum install -y docker-ce
```

docker-ce为社区免费版本。稍等片刻，docker即可安装成功。



## 2.3.启动docker

Docker应用需要用到各种端口，逐一去修改防火墙设置。非常麻烦，因此建议大家直接关闭防火墙！

启动docker前，一定要关闭防火墙后！！

启动docker前，一定要关闭防火墙后！！

启动docker前，一定要关闭防火墙后！！



```sh
# 关闭
systemctl stop firewalld
# 禁止开机启动防火墙
systemctl disable firewalld
```



通过命令启动docker：

```sh
systemctl start docker  # 启动docker服务

systemctl stop docker  # 停止docker服务

systemctl restart docker  # 重启docker服务
```



然后输入命令，可以查看docker版本：

```
docker -v
```

如图：

![](/img/image-20210418154704436.png )

## 2.4.配置镜像加速

docker官方镜像仓库网速较差，我们需要设置国内镜像服务：

参考阿里云的镜像加速文档：https://cr.console.aliyun.com/cn-hangzhou/instances/mirrors

```
sudo mkdir -p /etc/docker
sudo tee /etc/docker/daemon.json <<-'EOF'
{
 "registry-mirrors": ["https://a0yq40lf.mirror.aliyuncs.com"]
}
EOF
sudo systemctl daemon-reload
sudo systemctl restart docker
```

> 上面的镜像可能有问题奥，不行用下面的：
>
> - https://docker.1ms.run



# 3.CentOS7安装DockerCompose



## 3.1.下载

Linux下需要通过命令下载：

```sh
# 安装
curl -L https://github.com/docker/compose/releases/download/1.23.1/docker-compose-`uname -s`-`uname -m` > /usr/local/bin/docker-compose
```

如果下载速度较慢，或者下载失败，可以使用课前资料提供的docker-compose文件：

![](/img/image-20210417133020614.png )

上传到`/usr/local/bin/`目录也可以。



## 3.2.修改文件权限

修改文件权限：

```sh
# 修改权限
chmod +x /usr/local/bin/docker-compose
```





## 3.3.Base自动补全命令：

```sh
# 补全命令
curl -L https://raw.githubusercontent.com/docker/compose/1.29.1/contrib/completion/bash/docker-compose > /etc/bash_completion.d/docker-compose
```

如果这里出现错误，需要修改自己的hosts文件：

```sh
echo "199.232.68.133 raw.githubusercontent.com" >> /etc/hosts
```





# 4.Docker镜像仓库

搭建镜像仓库可以基于Docker官方提供的DockerRegistry来实现。

官网地址：https://hub.docker.com/_/registry



## 4.1.简化版镜像仓库

Docker官方的Docker Registry是一个基础版本的Docker镜像仓库，具备仓库管理的完整功能，但是没有图形化界面。

搭建方式比较简单，命令如下：

```sh
docker run -d \
    --restart=always \
    --name registry	\
    -p 5000:5000 \
    -v registry-data:/var/lib/registry \
    registry
```



命令中挂载了一个数据卷registry-data到容器内的/var/lib/registry 目录，这是私有镜像库存放数据的目录。

访问http://YourIp:5000/v2/_catalog 可以查看当前私有镜像服务中包含的镜像



## 4.2.带有图形化界面版本

使用DockerCompose部署带有图象界面的DockerRegistry，命令如下：

```yaml
version: '3.0'
services:
  registry:
    image: registry
    volumes:
      - ./registry-data:/var/lib/registry
  ui:
    image: joxit/docker-registry-ui:static
    ports:
      - 8080:80
    environment:
      - REGISTRY_TITLE=林羽私有仓库
      - REGISTRY_URL=http://registry:5000
    depends_on:
      - registry
```



## 4.3.配置Docker信任地址

我们的私服采用的是http协议，默认不被Docker信任，所以需要做一个配置：

```sh
# 打开要修改的文件
vi /etc/docker/daemon.json
# 添加内容：
"insecure-registries":["http://192.168.150.101:8080"]
# 重加载
systemctl daemon-reload
# 重启docker
systemctl restart docker
```



# 5. Docker 的操作

## **镜像：**

- 构建镜像：docker build
- 拉取镜像（docker pull xxxxxxx）：直接去官网看：https://hub.docker.com/  
- 查看镜像：docker images
- 删除镜像：docker rmi 
- 推送镜像到服务（禁止，除非你有镜像服务器）：docker push

- 保持镜像为一个压缩包：docker save
- 加载压缩包为镜像：docker load

## **容器：**

1. ### 容器外部

- 创建一个容器：
  - 参考指令：docker run ---name nignxly -p 80:80 -d nginx:指定版本号
  - 命令：
    - docker run：创建并运行一个容器
    - --name：给容器起一个名字，比如 **`nignxly `**
    - -p：将宿主机端口与容器端口映射，冒号左边的是 **宿主机端口**，右边的是 **容器端口**
    - -d：后台运行容器
    - nginx：镜像名称，后面可以跟 **`:镜像的版本号`**，如果只有一个这个镜像的话不加也行
- 删除容器：docker rm 容器名（不能删除运行中的容器，除非 -f 强删）
- 查看容器日志的命令：
  - docker logs
  - 添加 -f 参数可以持续查看日志
- 查看容器状态：
  - 查看**`运行中容器`**的状态：docker ps
  - 查看**`所有容器`**的状态：docker ps -a
- 启动容器：docker start 容器名
- 停止容器：docker stop 容器名
- 暂停容器：docker pause 容器名
- 唤醒容器：docker unpause 容器名
- 进入容器：dokcer exec -it 容器名 bash
- 退出容器：exit

2. ### 容器内部：

- 进入容器：dokcer exec -it nginxly bash
  - 命令解析：
    - docker exec：进入容器内部，执行一个命令
    - -it：给当前进入的容器创建一个标准输入、输出终端，允许我们与容器交互
    - nginxly：要进入的容器名称
    - bash：进入容器后执行的命令，bash是一个linux终端交互命令
- 进入容器后，就跟正常在操作系统一样操作
- 修改内容：难嗷~

```powershell
sed -i 's#Welcome to nginx#我个蛋啊#g' index.html
sed -i 's#<head>#<head><meta charset="utf-8">#g' index.html
```



3. ## 数据卷

> 为什么要有数据卷？
>
> - 容器与数据耦合的问题:
>   - **不便于修改**：当我们要修改Nginx的html内容时，需要进入容器内部修改，很不方便。
>   - **数据不可复用**：在容器内的修改对外是不可见的。所有修改对新创建的容器是不可复用的。
>   - **升级维护困难**：数据在容器内，如果要升级容器必然删除旧容器，所有数据都跟着删除了

**数据卷（volume）**：是一个虚拟目录，指向宿主机文件系统中的某个目录

、

### **基本操作语法：**

- docker volume [COMMAND]
  - **create：**创建一个 volume
  - **inspect：**显示一个或多个volume的信息
  - **ls：**列出所有的volume
  - **prune：**删除未使用的volume
  - **rm：**删除一个或多个指定的volume

docker volume 命令是数据卷操作，根据命令后跟随的 command 来确定下一步的操作；

- 创建一个数据卷，并查看数据卷在宿主机的目录位置
  - 创建数据卷：docker volume create html
  - 查看所有数据卷：dokcer volume ls
  - 查看数据卷详细信息卷：docker volume inspect html

### 挂载数据卷：

我们在创建容器时，可以通过 -v 参数来挂载一个数据卷到某个容器目录（没有这个数据卷会默认创建）

主要指令： -v html:/usr/share/nginx/html

（当然也可以自定义目录挂载：上述 `左侧的 html 切换为具体的文件全路径`）

- docker run --name ng -v html:/usr/share/nginx/html -p 80:80 -d nginx
-  进入html数据卷所在位置，并修改HTML内容
  - 查看html数据卷的位置：docker volume inspect html
  - #进入该目录：cd /var/lib/docker/volumes/html/_data
  -  修改文件（也可以直接修改）：vi index.html



# 6. Docker 容器启动配置参考

## 1. MySQL 配置启动

```powershell
docker run \
--name mysql8 \
-e MYSQL_ROOT_PASSWORD=123456   \
-p 3306:3306 \
-v /tmp/mysql/data:/var/lib/mysql      \
-v /tmp/mysql/conf:/etc/mysql/conf.d       \
-d  \
mysql
```

## 2. RabbitMQ 配置启动

```powershell
docker run \
 -e RABBITMQ_DEFAULT_USER=xiaoyu \
 -e RABBITMQ_DEFAULT_PASS=xiaoyu \
 -v mq-plugins:/plugins \
 --name mq \
 --hostname mq \
 -p 15672:15672 \
 -p 5672:5672 \
 -d \
 rabbitmq
```

