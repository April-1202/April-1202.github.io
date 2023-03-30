# docker的安装使用教程



这篇文章提供了docker的安装使用教程

<!--more-->


## 一 什么是docker

Docker是一种开源的容器化平台，可以让开发者将应用程序及其依赖项打包到一个轻量级、可移植的容器中，然后在不同的操作系统上运行



## 二 linux环境下的docker的安装教程

安装地址：https://docs.docker.com/engine/install/centos/

### 1 先决条件
确定你是CentOS7及以上版本 （ cat /etc/redhat-release ）


### 2 卸载旧版本
在尝试安装新版本之前卸载任何此类旧版本以及相关的依赖项，
如果没有安装过docker执行此命令也没有关系
```卸载命令
 sudo yum remove docker \
                  docker-client \
                  docker-client-latest \
                  docker-common \
                  docker-latest \
                  docker-latest-logrotate \
                  docker-logrotate \
                  docker-engine

```

### 3 yum安装gcc相关（安装过gcc可跳过此步）
  yum -y install gcc
  yum -y install gcc-c++

### 4 安装需要的软件包
  yum install -y yum-utils

### 5 设置stable镜像仓库
  yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo<br>
  这行命令意思是每次下载优先从阿里云的仓库中下载

### 6 更新yum软件包索引
yum makecache fast

### 7 开始安装docker ce
yum install docker-ce docker-ce-cli containerd.io docker-compose-plugin<br>
注：docker ce 为社区免费版  docker ee 为企业版

### 8 启动docker
systemctl start docker

### 9 测试docker是否安装成功
docker version  或  docker run hello-world


