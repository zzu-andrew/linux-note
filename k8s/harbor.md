

在云原生环境下，在容器镜像中打包了所要运行软件的所有内容。因此，对容器镜像的管理在整个云原生应用的开发、测试、部署和管理中，是一个非常重要的组成部分，也是Harbor主要解决的问题。


## 容器运行时

`Linux` 提供了命名空间和控制组两大系统功能，它们是容器的基础。但是，要把进程运行在容器中，还需要有便捷的`SDK`或命令来调用`Linux`的系统功能，从而创建出容器。容器的运行时（`runtime`）就是容器进程运行和管理的工具。
容器运行时分为低层运行时和高层运行时，功能各有侧重。低层运行时主要负责运行容器，可在给定的容器文件系统上运行容器的进程；高层运行时则主要为容器准备必要的运行环境，如容器镜像下载和解压并转化为容器所需的文件系统、创建容器的网络等，然后调用低层运行时启动容器。

![[Pasted image 20250205111925.png]]


## `CRI` 和 `CRI-O`


![[Pasted image 20250205113558.png]]

*OCI镜像规范*

OCI 定义的镜像包括4个部分：镜像索引（Image Index）、清单 （Manifest）、配置（Configuration）和层文件（Layers）。

*本地计算机上容器生命周期变化*

.本地计算机上容器生命周期变化
![[image-2025-02-05-14-54-16-661.png]]

## 功能和架构

### 核心功能

作为云原生制品仓库服务，Harbor的核心功能是存储和管理Artifact(制品)。Harbor允许用户用命令行工具对容器镜像及其他Artifact进行推送和拉取，并提供了图形管理界面帮助用户查阅和删除这些Artifact。

#### 访问控制

Harbor 提供了“项目”（project）的概念，每个项目都对应一个和项目名相同的命名空间（namespace）来保存Artifact，各个命名空间都是彼此独立的授权单元，将 Artifact 隔离开来。当使用 Docker 等命令行工具向Harbor推送和拉取镜像等Artifact时，这个命名空间也是URI的一个组成部分。 用户要对项目中的Artifact进行读写，就首先要被管理员添加为项目的成员，具体的权限由成员的角色决定。


## 安装

*安装先决条件*

https://goharbor.io/docs/2.12.0/install-config/installation-prereqs/[harbor安装最低配置要求]

*去下载页下载harbor*

https://github.com/goharbor/harbor/releases[harbor下载页]

harbor一般会发布两个版本，一个在线安装版本，一个离线安装版本。

- 在线安装版本：`harbor-offline-installer-v2.12.2.tgz`
- 离线安装版本：`harbor-online-installer-v2.12.2.tgz`

### harbor.yml.tmpl关键配置

- `hostname` ： 用来配置Harbor服务的网络访问地址。可以将其配置成当前安装环境的IP地址和主机域名(FQDN)。
- `HTTP&HTTPS` : 配置Harbor的网络访问协议，默认是HTTPS。注意：如果选择安装Notary组件时修将Harbor的网络访问协议配置为HTTPS。启动HTTPS需要使用证书，Harbor提供了证书的自动生成工具，可以通过以下命令来生成。

```bash
$ docker run -v /:/hostfs goharbor/prepare:v2.0.0 gencert -p /path/to/internal/tls/cert
```

下载完离线包之后会得到如下几个文件

```bash
[root@k8smaster-ims harbor]# ls
LICENSE  common.sh  harbor.v2.12.2.tar.gz  harbor.yml.tmpl  install.sh  prepare
```

在配置好yaml之后，只需要执行install.sh即可实现harbor的安装。

### helm chart安装harbor

在使用helm安装时需要将harbor的helm地址添加到helm仓库中。

```bash
# 添加harbor helm地址
helm repo add harbor https://helm.goharbor.io
# 安装harbor
help install -name my-release harbor/harbor
# 卸载harbor
helm delete --purge -my-release
# 卸载harbor
helm uninstall my-release
```


### 安装成功界面

.安装成功命令行界面
![[image-2025-02-06-10-50-21-360.png]]


.安装成功web界面
![[image-2025-02-06-10-52-48-289.png]]

.登陆成功界面
![[image-2025-02-06-10-53-17-453.png]]



## harbor编译

1. 下载harbor源码

.查看harbor当前最新版本
![[image-2025-02-12-19-32-19-226.png]]

```bash
git clone https://github.com/goharbor/harbor.git
cd harbor
# 这里将版本切换到稳定版本进行打包编译
git checkout v2.12.0
```

```bash
# 使用make进行镜像打包，命令如下
make build -e DEVFLAG=false COMPILETAG=compile_golangimage VERSIONTAG=v2.12.0
```


```bash
[root@node1 harbor]# docker images | grep harbor |grep v2.12.0
goharbor/harbor-exporter                             v2.12.0            aa2b04d6bd6e   41 seconds ago       96.2MB
goharbor/redis-photon                                v2.12.0            d4add902ed22   59 seconds ago       165MB
goharbor/harbor-registryctl                          v2.12.0            a4435d398f5c   32 seconds ago   134MB
goharbor/registry-photon                             v2.12.0            3e1747aa237e   2 minutes ago        78.1MB
goharbor/nginx-photon                                v2.12.0            14a28d36b486   3 minutes ago        45MB
goharbor/harbor-log                                  v2.12.0            9caba2b11401   3 minutes ago        159MB
goharbor/harbor-jobservice                           v2.12.0            1b6f469183ad   4 minutes ago        241MB
goharbor/harbor-core                                 v2.12.0            d16215207e8f   5 minutes ago        207MB
goharbor/harbor-portal                               v2.12.0            43081afe66fb   6 minutes ago        53.6MB
goharbor/harbor-db                                   v2.12.0            6a355218db09   8 minutes ago        225MB
goharbor/prepare                                     v2.12.0            6eb9a65377f6   9 minutes ago        254MB
```

## harbor安装

在下面的目录中替换自己的harbor安装包

```bash
[root@k8smaster-73 amd64]# pwd
/home/db/packages/YsP-1/BusinessConfig/harbor/image/amd64
[root@k8smaster-73 amd64]# ls
harbor.v2.10.2.tar
```

然后再shell目录下执行 `install_harbor.sh` 安装命令进行harbor的部署

```bash
[root@k8smaster-73 shell]# pwd
/home/db/packages/YsP-1/BusinessConfig/harbor/shell
[root@k8smaster-73 shell]# ls
clean_harbor.sh  connect_remote_harbor.sh  harbor_pg_recover.sh     install_harbor.sh  upgrade_harbor.sh
common.sh        harbor_backup_crontab.sh  harbor_redis_recover.sh  scale_harbor.sh
```

如果是升级执行 `upgrade_harbor.sh` 脚本。

## 常见问题

### 在丢失secret key的情况下删除已签名的镜像

Harbor的镜像签名功能是镜像安全相关功能的重要一环，由Notary 提供。已签名的镜像需要得到Notary的验证和授权才能被删除，如果用户丢失了Notary的secret key，则无法再删除已签名的镜像。在这种情况下，管理员是可以绕开限制将其删除。

### 丢失了系统管理员admin的密码

- 使用客户端连接Harbor的数据库，并选择Harbor数据库：

```bash
$ docker exec -it harbor-db
$ psql -U postgres
postgres=# \c registry;
```

- 执行如下命令，重置admin账号：

```bash
registry=# select * from harbor_user update harbor_user set salt = '', password = '' where user_id = 1;
```

- 再次重启Harbor服务：重新设置密码就行了









































