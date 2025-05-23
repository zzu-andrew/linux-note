
## 第一阶段：基础知识

### **了解容器化技术**
- **什么是容器？**：容器是轻量级、可移植的运行环境，包含应用程序及其所有依赖项。容器与虚拟机不同，它们共享主机操作系统的内核，因此更轻量且启动更快。
- **容器化的优势**：隔离性、可移植性、资源利用率高、开发和生产环境一致性等。
- **Docker 的作用**：Docker 是一个开源平台，用于构建、部署和管理容器化的应用程序。

### **安装 Docker**
- **在 Linux 上安装 Docker**：
- 使用官方文档中的命令行安装方法，我这里使用的是ubuntu环境，因此后面的测试实验都是在ubuntu环境下进行。
环境安装可以参考官方安装指导文档： https://docs.docker.com/engine/install/[Docker 官方安装指南]

Docker 架构图
![[c8116066bdbf295a7c9fc25b87755dfe.jpg]]

###  **熟悉 Docker 基本命令**


| 命令                                   | 说明                                   |
| ------------------------------------ | ------------------------------------ |
| **`docker --version`**               | 检查 Docker 版本。                        |
| **`docker info`**                    | 查看 Docker 系统信息。                      |
| **`docker run`**                     | 启动一个新的容器。                            |
| **`docker ps`**                      | 列出正在运行的容器。                           |
| **`docker ps -a`**                   | 列出所有容器（包括已停止的）。                      |
| **`docker stop <container_id>`**     | 停止一个正在运行的容器。                         |
| **`docker rm <container_id>`**       | 删除一个已停止的容器。                          |
| **`docker images`**                  | 列出本地镜像。                              |
| **`docker rmi <image_id>`**          | 删除一个本地镜像。                            |
| **`docker pull <image_name>`**       | 从 Docker Hub 拉取镜像。                   |
| **`docker build -t <image_name> .`** | 从 Dockerfile 构建镜像。                   |
| **`docker stats`**                   | 查看容器的CPU等资源使用情况。                     |
| **`docker logs`**                    | 查看容器的日志。                             |
| **`docker exec`**                    | 进入到容器内部执行命令                          |
| **`docker history`**                 | 查看镜像的打包过程                            |
| **`docker volume`**                  | 查看docker生成的持久化挂载卷，这些卷能保证容器重启之后数据依然存在 |

#### `docker commit`

`docker commit` 命令用于将一个容器中的更改提交为一个新的镜像。

这个命令特别是在国内特别有用，国内开发打包镜像比较麻烦，因此可以先将需要的镜像下载到本地，然后运行之后，替换需要替换的文件，然后使用 `docker commit` 命令来提交一个容器的更改为一个新的镜像。

.eg
```bash
docker commit -m "commit message" -a "author" <container_id> <image_name>
```


#### `docker volume`

管理docker生成的卷

- `docker volume ls` 查看卷
- `docker volume inspect` 查看卷的详细信息
- `docker volume rm` 删除一个卷
- `docker volume create` 创建一个卷
- `docker volume prune` 删除所有未使用的卷

如果 `docker compose` 停止时，想要删除相关的卷，可以使用 `docker-compose down --volumes`


### `docker build`

```bash
# -t指定镜像名称和标签， .表示在当前目录下构建镜像
docker build -f Dockerfile -t myappimage:v1.0.0.0 .
```

#### `docker images`

用于查看镜像列表信息

#### `docker image`

- `docker image history`

对单个镜像的管理，比如查看镜像的详细打包过程

```bash
andrew in ~ λ docker image history redis
IMAGE          CREATED        CREATED BY                                       SIZE      COMMENT
43724892d6db   2 months ago   CMD ["redis-server"]                             0B        buildkit.dockerfile.v0
<missing>      2 months ago   EXPOSE map[6379/tcp:{}]                          0B        buildkit.dockerfile.v0
<missing>      2 months ago   ENTRYPOINT ["docker-entrypoint.sh"]              0B        buildkit.dockerfile.v0
<missing>      2 months ago   COPY docker-entrypoint.sh /usr/local/bin/ # …   661B      buildkit.dockerfile.v0
<missing>      2 months ago   WORKDIR /data                                    0B        buildkit.dockerfile.v0
<missing>      2 months ago   VOLUME [/data]                                   0B        buildkit.dockerfile.v0
<missing>      2 months ago   RUN /bin/sh -c mkdir /data && chown redis:re…   0B        buildkit.dockerfile.v0
<missing>      2 months ago   RUN /bin/sh -c set -eux;   savedAptMark="$(a…   38.1MB    buildkit.dockerfile.v0
<missing>      2 months ago   ENV REDIS_DOWNLOAD_SHA=4ddebbf09061cbb589011…   0B        buildkit.dockerfile.v0
<missing>      2 months ago   ENV REDIS_DOWNLOAD_URL=http://download.redis…   0B        buildkit.dockerfile.v0
<missing>      2 months ago   ENV REDIS_VERSION=7.4.2                          0B        buildkit.dockerfile.v0
<missing>      2 months ago   RUN /bin/sh -c set -eux;  savedAptMark="$(ap…   4.12MB    buildkit.dockerfile.v0
<missing>      2 months ago   ENV GOSU_VERSION=1.17                            0B        buildkit.dockerfile.v0
<missing>      2 months ago   RUN /bin/sh -c set -eux;  apt-get update;  a…   5.08kB    buildkit.dockerfile.v0
<missing>      2 months ago   RUN /bin/sh -c set -eux;  groupadd -r -g 999…   4.3kB     buildkit.dockerfile.v0
<missing>      2 months ago   # debian.sh --arch 'amd64' out/ 'bookworm' '…   74.8MB    debuerreotype 0.15
```

- `docker image inspect`

查看docker镜像详细的分成信息，以及镜像具体的组成方式。

```bash
andrew in ~ λ docker image --help
Usage:  docker image COMMAND

Manage images

Commands:
  build       Build an image from a Dockerfile
  history     Show the history of an image
  import      Import the contents from a tarball to create a filesystem image
  inspect     Display detailed information on one or more images
  load        Load an image from a tar archive or STDIN
  ls          List images
  prune       Remove unused images
  pull        Download an image from a registry
  push        Upload an image to a registry
  rm          Remove one or more images
  save        Save one or more images to a tar archive (streamed to STDOUT by default)
  tag         Create a tag TARGET_IMAGE that refers to SOURCE_IMAGE
```

#### `docker inspect`

#### `docker ps`

- `docker ps -s`

查看容器读写层大小和原先镜像只读层大小，比如 `21.2MB (virtual 585MB)` 代表容器读写层大小为 21.2MB，原先镜像只读层大小为 585MB

```bash
andrew in ~ λ docker ps -s
CONTAINER ID   IMAGE                                         COMMAND                   CREATED          STATUS          PORTS                                         NAMES                       SIZE
743e978d957b   redis:latest                                  "docker-entrypoint.s…"   44 minutes ago   Up 44 minutes   0.0.0.0:6380->6379/tcp, [::]:6380->6379/tcp   redis-slave1                0B (virtual 117MB)
7cd38da53362   redis:latest                                  "docker-entrypoint.s…"   44 minutes ago   Up 44 minutes   0.0.0.0:6379->6379/tcp, [::]:6379->6379/tcp   redis-master                0B (virtual 117MB)
c1f54aad72e0   redis:latest                                  "docker-entrypoint.s…"   44 minutes ago   Up 44 minutes   0.0.0.0:6381->6379/tcp, [::]:6381->6379/tcp   redis-slave2                0B (virtual 117MB)
9d852ccb43c6   nginx:latest                                  "/docker-entrypoint.…"   5 hours ago      Up 5 hours      0.0.0.0:8080->80/tcp, [::]:8080->80/tcp       nginx_web_1                 1.09kB (virtual 192MB)
f2764175ad8c   grafana/grafana                               "/run.sh"                 6 hours ago      Up 6 hours                                                    grafana                     21.2MB (virtual 585MB)
```


### 镜像

#### *获取镜像*

`docker pull` 命令用来从网络 "镜像仓库" 中拉取镜像到本地

`docker pull` 是 Docker 中用于从镜像仓库（如 Docker Hub 或私有仓库）拉取镜像的命令。通过 `docker pull`，您可以下载指定的镜像或整个仓库中的所有标签化的镜像。

使用 `docker pull --help` 查看下docker官方给出的命名说明

```bash
[root@k8smaster-211 ~]# docker pull --help

Usage:  docker pull [OPTIONS] NAME[:TAG|@DIGEST]

Pull an image or a repository from a registry

Options:
  -a, --all-tags                Download all tagged images in the repository
      --disable-content-trust   Skip image verification (default true)
      --platform string         Set platform if server is multi-platform capable
  -q, --quiet                   Suppress verbose output
```

.eg 拉取特定版本的镜像
```bash
docker pull [OPTIONS] NAME[:TAG|@DIGEST]
```

- **`NAME`**：镜像的名称，通常是仓库名称。例如，`alpine`、`nginx`、`mysql` 等。
- **`TAG`**：镜像的标签，表示镜像的版本。默认标签是 `latest`。例如，`nginx:1.21.6` 表示 Nginx 1.21.6 版本的镜像。
- **`DIGEST`**：镜像的内容哈希值，确保拉取的是特定的镜像版本。例如，`nginx@sha256:abc123...`。

- **OPTIONS**: 选项

.eg 默认拉取最新版本的镜像 nginx:latest
```bash
docker pull nginx
```

.eg 拉取指定版本的nginx
```bash
docker pull nginx:1.21.6
```

.eg 按照哈希值拉取nginx
```bash
docker pull nginx@sha256:abc123...
```

##### `-a, --all-tags`

- **描述**：下载仓库中所有带有标签的镜像。
- **用法**：如果您想一次性拉取某个仓库中的所有版本，可以使用这个选项。

##### `--disable-content-trust`

- **描述**：跳过镜像验证，默认情况下 Docker 会启用内容信任（Content Trust），确保拉取的镜像是由官方签名的。如果您不关心镜像的安全性或正在使用不受信任的仓库，可以禁用此功能。
- **用法**：在某些情况下，您可能需要禁用内容信任以拉取未经签名的镜像。

.eg 拉取一个不受信任的镜像
```bash
docker pull --disable-content-trust muApp
```

##### `--platform string`

- **描述**：指定目标平台，适用于多平台镜像。Docker 支持多种架构（如 `linux/amd64`、`linux/arm64`、`windows/amd64` 等）。如果您在一个平台上运行 Docker，但需要为另一个平台拉取镜像，可以使用此选项。

- **用法**：指定目标平台的格式为 `<os>/<arch>`。

.eg 在 x86_64 架构的 Linux 主机上拉取 ARM64 版本的 Nginx 镜像：
```bash
docker pull --platform linux/arm64 nginx
```

##### `-q, --quiet`

- **描述**：抑制详细输出，只显示镜像 ID。当您不需要看到详细的拉取过程时，可以使用此选项来减少输出信息。

```bash
docker pull -q nginx
```

#### 查看镜像信息

`docker images` 命令用于列出本地 Docker 主机上的所有镜像。

```bash
[root@k8smaster-211 ~]# docker images --help

Usage:  docker images [OPTIONS] [REPOSITORY[:TAG]]

List images

Options:
  -a, --all             Show all images (default hides intermediate images)
      --digests         Show digests
  -f, --filter filter   Filter output based on conditions provided
      --format string   Pretty-print images using a Go template
      --no-trunc        Don't truncate output
  -q, --quiet           Only show numeric IDs
You have new mail in /var/spool/mail/root
```

- **`REPOSITORY`**：指定要列出的镜像仓库名称。如果不提供，默认列出所有仓库的镜像。
- **`TAG`**：指定要列出的镜像标签。如果不提供，默认列出所有标签的镜像。

.eg 列出所有本地镜像：
```bash
docker images
```

.eg 列出特定仓库的所有镜像（包括不同标签）：
```bash
docker images nginx
```

.eg 列出特定仓库和标签的镜像：
```bash
docker images nginx:1.21.6
```

##### `-a, --all`

- **描述**：显示所有镜像，包括中间层镜像（intermediate images）。默认情况下，`docker images` 只显示顶层镜像，即那些没有被其他镜像作为基础层使用的镜像（Dokcerfile部分会进行说明）。
- **用法**：当您想查看所有镜像，包括构建过程中生成的中间层镜像时，可以使用此选项。

```bash
docker images -a
```

##### `--digests`

- **描述**：显示镜像的内容哈希值（digest）。这有助于确保拉取的镜像是特定版本，而不是最新的标签。
- **用法**：当您需要验证镜像的完整性或确保使用的是特定版本时，可以使用此选项。

```bash
docker images --digests
```

##### `-f, --filter filter`

- **描述**：根据指定的条件过滤输出。常用的过滤条件包括 `dangling`、`label`、`before` 和 `since`。
- **常用过滤条件**：
- `dangling=true`：只显示悬空镜像（即没有标签且未被任何容器使用的镜像）。
- `label=key=value`：根据镜像的标签进行过滤。
- `before=image_name`：显示创建时间早于指定镜像的镜像。
- `since=image_name`：显示创建时间晚于指定镜像的镜像。

```bash
docker images -f dangling=true
```

.eg 根据标签过滤镜像
```bash
docker images -f label=version=1.0
```

##### `--format string`

- **描述**：使用 Go 模板格式化输出。您可以自定义输出的列和顺序，以便更方便地查看所需信息。
- **常用模板变量**：
- `{{.ID}}`：镜像 ID
- `{{.Repository}}`：仓库名称
- `{{.Tag}}`：标签
- `{{.Digest}}`：内容哈希值
- `{{.CreatedSince}}`：创建时间（相对）
- `{{.CreatedAt}}`：创建时间（绝对）
- `{{.Size}}`：镜像大小

```bash
docker images --format "{{.ID}}: {{.Repository}}"
```

##### `--no-trunc`

- **描述**：不截断输出，显示完整的镜像 ID 和标签。默认情况下，Docker 会截断长字符串以适应终端宽度。
- **用法**：当您需要查看完整的镜像 ID 或标签时，可以使用此选项。

```bash
docker images --no-trunc
```

##### `-q, --quiet`

- **描述**：仅显示镜像的短 ID（前 12 个字符），适合用于脚本或自动化任务。
- **用法**：简化输出，方便与其他命令结合使用。

```bash
docker images -q
```

#### 为镜像打标签

`docker tag` 命令用于为镜像打标签。

```bash
[root@k8smaster-211 ~]# docker tag --help

Usage:  docker tag SOURCE_IMAGE[:TAG] TARGET_IMAGE[:TAG]

Create a tag TARGET_IMAGE that refers to SOURCE_IMAGE
```

`docker tag` 其实就是给镜像起个别名，经过 `docker images` 查看经过docker tag处理的镜像和原先的镜像ID是一样的。

如果想查看镜像的详细信息可以使用docker images进行查看。

#### 搜索镜像

`docker search` 命令用于搜索 Docker Hub 上的镜像。

详细信息可以参考 `docker search --help`

#### 删除镜像

使用命令 `docker rmi IMAGE [IMAGE ...]` 可以将指定镜像删除，IMAGE可以替换成对应镜像文件的ID。 如果前期镜像有多个标签(经过docker tag处理)，删除时会先删除标签，直到删除最后一个标签时，镜像会跟着一起被删除

> 删除镜像时，如果镜像有容器在使用，需要先停止所有使用这个镜像的容器，才能删除镜像。，当然如果你想强制删除镜像，可以使用 `docker rmi -f <image_id>` 和linux命令一样加上 -f 参数表示强制删除。但是使用强制删除会有一个遗留问题，那就是原来被强制删除的镜像会改变一个新的镜像ID之后继续存在系统中，因此正确的做法是停止所有依赖该镜像的容器，然后再删除镜像。

```bash
[root@k8smaster-211 ~]# docker rmi --help

Usage:  docker rmi [OPTIONS] IMAGE [IMAGE...]

Remove one or more images

Options:
  -f, --force      Force removal of the image
  --no-prune   Do not delete untagged parents
```

#### 创建镜像

镜像的创建可以分为三种常见情况：

[cols="4*", options="header"]
|===
| 方法 | 适用场景 | 优点 | 缺点

| **使用 `Dockerfile` 构建镜像**
| 需要定义可重复、可维护的镜像构建过程
| 可重复性、可维护性、灵活性
| 需要编写 `Dockerfile`

| **使用 `docker commit` 从容器创建镜像**
| 快速保存容器的临时修改
| 快速便捷、灵活性
| 不可重复性、镜像臃肿、维护困难

| **使用 `docker save` 和 `docker load` 导入/导出镜像**
| 在不同机器之间传输镜像或备份/恢复
| 方便传输、备份和恢复、适合离线环境
| 手动操作、不适用于频繁更新
|===

##### **使用 `Dockerfile` 构建镜像**

.适用场景：
****
- 您希望定义一个可重复、可维护的镜像构建过程。
- 您需要确保镜像在不同环境中的一致性。
- 您希望团队成员能够轻松理解和复现镜像的构建步骤。
****

1. **编写 `Dockerfile`**：
`Dockerfile` 是一个文本文件，包含一系列指令，用于定义如何构建 Docker 镜像。每个指令都会在镜像中创建一个新的层。以下是一个简单的 `Dockerfile` 示例，用于创建一个包含 Nginx 和自定义配置的镜像：

.Dockerfile 示例
```Dockerfile
# 使用官方的 Nginx 镜像作为基础镜像
FROM nginx:latest

# 设置工作目录
WORKDIR /usr/share/nginx/html

# 将本地的 HTML 文件复制到镜像中
COPY ./html/* .

# 暴露 80 端口
EXPOSE 80

# 设置默认命令（可选）
CMD ["nginx", "-g", "daemon off;"]
```

2. **准备必要的文件**：
确保您的项目目录中包含所有需要的文件。例如，假设您有一个 `html` 目录，其中包含静态网页文件（如 `index.html`），并且您希望将这些文件复制到 Nginx 的默认 Web 根目录中。

```bash
.
├── Dockerfile
└── html
   └── index.html
```

3. **构建镜像**：

使用 `docker build` 命令从 `Dockerfile` 构建镜像。您可以为镜像指定一个名称和标签。

```bash
docker build -t my_nginx_image:1.0 .
```

- **`-t`**：指定镜像的名称和标签（格式为 `name:tag`）。如果没有指定标签，默认标签是 `latest`。
- **`.`**：表示 `Dockerfile` 所在的当前目录。Docker 会在这个目录中查找 `Dockerfile`，并将其作为构建上下文。

4. **验证镜像**：
构建完成后，您可以使用 `docker images` 命令查看新创建的镜像。

```bash
docker images
```

您应该能看到类似以下的输出：

```plaintext
REPOSITORY          TAG       IMAGE ID       CREATED         SIZE
my_nginx_image      1.0       abc123def456   2 minutes ago   133MB
```

##### *基于已有镜像的容器创建*

`docker commit` 命令用于将一个容器转换为镜像。其命令格式如下：

```bash
[root@k8smaster-211 ~]# docker commit --help

Usage:  docker commit [OPTIONS] CONTAINER [REPOSITORY[:TAG]]

Create a new image from a container's changes

Options:
  -a, --author string    Author (e.g., "John Hannibal Smith <hannibal@a-team.com>")
  -c, --change list      Apply Dockerfile instruction to the created image
  -m, --message string   Commit message
  -p, --pause            Pause container during commit (default true)
```

- **`CONTAINER`**：要提交的容器 ID 或名称。
- **`REPOSITORY`**：新镜像的仓库名称。如果不指定，默认会创建一个无标签的镜像。
- **`TAG`**：新镜像的标签。如果不指定，默认标签是 `latest`。

.eg 从容器 `my_container` 创建一个名为 `my_image:1.0` 的新镜像：
```bash
# 注意这里是重新创建一个镜像，而tag命令只是给一个别名
# 先使用docker ps 查看运行的容器，然后由运行中的容器创建
docker commit my_container my_image:1.0
```

.eg 从容器 `my_container` 创建一个无标签的新镜像：
```bash
docker commit my_container
```


##### `-a, --author string`

- **描述**：指定新镜像的作者信息，通常包括姓名和电子邮件地址。这有助于记录谁创建了该镜像。
- **用法**：提供一个字符串作为作者信息。

```bash
docker commit -a "John Hannibal Smith <hannibal@a-team.com>" my_container my_image:1.0
```

##### `-c, --change list`

- **描述**：应用 Dockerfile 指令到新创建的镜像中。这允许您在提交时添加额外的配置或修改。常用的指令包括 `CMD`、`ENTRYPOINT`、`ENV`、`EXPOSE`、`LABEL`、`USER`、`WORKDIR` 和 `ONBUILD`。
- **用法**：提供一个或多个 Dockerfile 指令，每个指令之间用逗号分隔。

.g添加环境变量并设置工作目录：
```bash
docker commit -c "ENV MY_VAR=value" -c "WORKDIR /app" my_container my_image:1.0
```

#####  `-m, --message string`

- **描述**：为提交操作添加一个描述性消息。这有助于记录为什么创建了这个新镜像，类似于 Git 提交的消息。
- **用法**：提供一个字符串作为提交消息。

.eg 添加提交消息：
```bash
docker commit -m "Added new feature X" my_container my_image:1.0
```

##### `-p, --pause`

- **描述**：在提交过程中暂停容器。默认情况下，Docker 会在提交时暂停容器，以确保捕获容器的当前状态。如果您不希望暂停容器，可以使用此选项将其关闭。
- **用法**：默认值为 `true`，即暂停容器。如果不想暂停容器，可以传递 `--pause=false`。

.eg 不暂停容器进行提交：
```bash
docker commit --pause=false my_container my_image:1.0
```

- **避免频繁使用 `docker commit`**：虽然 `docker commit` 可以快速保存容器的状态，但它并不是最佳的镜像构建方式。推荐使用 Dockerfile 来定义镜像的构建过程，这样可以确保镜像的一致性和可重复性。
- **镜像大小问题**：每次使用 `docker commit` 都会创建一个新的镜像层，这可能会导致镜像变得臃肿。因此，建议定期清理不再需要的镜像，以节省磁盘空间。
- **安全性考虑**：确保在提交镜像时不会包含敏感信息（如密码、API 密钥等）。最好将这些信息作为环境变量或通过 Docker Secrets 管理。

##### **使用 `docker save` 和 `docker load` 导入/导出镜像**

- 您已经有本地的镜像文件（例如从其他机器导出的 `.tar` 文件），需要将其导入到本地 Docker 主机中。
- 您需要在不同机器之间传输镜像，或者备份和恢复镜像。

**导出镜像**：
使用 `docker save` 命令将本地镜像导出为 `.tar` 文件。

```bash
docker save -o my_image.tar my_image:1.0
```

- **`-o`**：指定输出文件的路径和名称。
- **`my_image:1.0`**：要导出的镜像名称和标签。

**导入镜像**：

使用 `docker load` 命令将 `.tar` 文件导入到本地 Docker 主机中。

```bash
docker load -i my_image.tar
```

- **`-i`**：指定输入文件的路径和名称。

#### 上传镜像到 Docker Hub

Docker Hub 是一个公共的镜像仓库，您可以在这里上传和分享您的镜像。不过前提是需要登录到 Docker Hub。上传镜像使用命令 `docker push`

```bash
[root@k8smaster-211 ~]# docker push --help

Usage:  docker push [OPTIONS] NAME[:TAG]

Push an image or a repository to a registry

Options:
      --disable-content-trust   Skip image signing (default true)
```

```bash
docker push my_image:1.0
```

### 容器

从开头的Docker架构图中能够看，如果把镜像和容器联系起来，那么镜像就是模板，容器就是实例。类比linux上的进程和可执行文件之间的关系，那么容器就是进程，而镜像就是可执行文件，同一个可执行程序可以创建多个进程，同样同一个镜像可以创建多个容器。

#### 创建容器

`docker create` 命令用于创建一个新的容器，但不启动它。与 `docker run` 不同，`docker create` 只会准备容器并生成一个容器 ID，而不会立即运行容器。这在某些场景下非常有用，例如您希望在启动前配置容器、检查容器的状态或设置网络和卷等资源。

```bash
docker create [OPTIONS] IMAGE [COMMAND] [ARG...]
```

- **`IMAGE`**：要使用的镜像名称或 ID。
- **`COMMAND`**：可选的命令，覆盖镜像中定义的默认命令（即 `CMD` 或 `ENTRYPOINT`）。
- **`ARG...`**：传递给命令的参数。

```bash
# 创建容器但不启动
docker create -it --name my_container my_image:1.0

# 进入容器进行配置
docker start -ai my_container

# 启动容器
docker start my_container
```

在启动容器之前，您可以使用 `docker inspect` 命令检查容器的配置，确保一切设置正确。

```bash
# 创建容器
docker create --name my_container -e MY_VAR=value my_image:1.0

# 检查容器配置
docker inspect my_container
```

.`docker create` 与 `docker run` 的区别

[cols="3*", options="header"]
|===
| 命令 | 描述 | 使用场景

| `docker create`
| 创建容器但不启动
| 适合在启动前进行配置、检查或预分配资源

| `docker run`
| 创建并启动容器
| 适合直接启动容器并立即使用
|===

#### 获取容器日志

`docker logs` 命令用于获取容器的日志。您可以通过指定容器 ID 或名称来获取日志。

```bash
docker logs my_container
```

#### 停止容器

`docker stop` 命令用于停止一个或多个正在运行的容器， 如果不指定`-t` 参数，默认等待 10 秒，如果容器在 10 秒内没有停止，则强制停止。

```bash
[root@k8smaster-211 ~]# docker stop --help

Usage:  docker stop [OPTIONS] CONTAINER [CONTAINER...]

Stop one or more running containers

Options:
  -t, --time int   Seconds to wait for stop before killing it (default 10)
```

.eg 停止容器：
```bash
docker stop e67
# docker ps 只能查看到运行中的容器，在停止之后，需要使用docker ps -a -q 查看处于通知状态的容器
docker ps -a -q
# 如果想重新启动停止之后的容器，可以使用docker restart [containerd id]
```

#### 进入容器内部

一般容器运行需要在后台运行，用户如果需要查看容器内部的信息，就需要进入容器内部。

```bash
docker exec -it my_container bash
# -i 表示交互式，-t 表示分配一个伪终端，bash 为要进入的容器中的命令，可以替换为需要的命令。
# 如果需要进入一个正在运行的容器，可以使用
docker attach <container_id>
```

> 多个窗口使用attach命令时，所有的窗口会同步显示，如果某个窗口因为命令执行阻塞了，其他窗口也无法执行操作了。

#### 删除容器

[aource, bash]
```
Usage:  docker rm [OPTIONS] CONTAINER [CONTAINER...]

Remove one or more containers

Aliases:
  docker container rm, docker container remove, docker rm

Options:
  -f, --force     Force the removal of a running container (uses SIGKILL)
  -l, --link      Remove the specified link
  -v, --volumes   Remove anonymous volumes associated with the container
```

- `-f, --force`

强制终止并删除一个正在运行中的容器

- `-l, --link`

删除容器的链接，但是保留容器本身

- `-v, --volume`

删除容器挂载的数据卷

#### 导入和导出容器

##### `docker export`


`docker export` 命令用于将 Docker 容器的文件系统导出为一个 tar 归档文件。不管这个容器是否处于运行状态。这个命令会捕获容器在运行时的文件系统状态，但不会包括容器的元数据，如网络配置、卷（volumes）、重启策略等。因此，如果你需要保存完整的容器状态，你应该考虑使用 `docker commit` 来创建一个新的镜像，或者使用 `docker save` 来保存镜像。

```bash
Usage:  docker export [OPTIONS] CONTAINER

Export a container's filesystem as a tar archive

Aliases:
  docker container export, docker export

Options:
  -o, --output string   Write to a file, instead of STDOUT
```


你可以通过指定 `-o` 或 `--output` 选项来直接将输出写入到一个文件中，而不是标准输出（stdout）。如果不指定该选项，tar 流将会被输出到标准输出，通常你会将其重定向到一个文件中。

例如，要将名为 `my_container` 的容器导出到一个名为 `my_container_backup.tar` 的文件中，你可以执行以下命令：

```bash
docker export -o my_container_backup.tar my_container
```

或者，如果你不想使用 `-o` 选项，可以使用重定向操作符 `>`：

```bash
docker export e81 > my_container_backup.tar
```

请确保你有足够的磁盘空间来保存 tar 文件，并且考虑到没有压缩，文件可能会比较大。如果你需要压缩归档，可以在导出过程中使用 gzip 或其他工具进行管道处理。例如：

```bash
docker export my_container | gzip > my_container_backup.tar.gz
```

##### 导入容器

```bash
Usage:  docker import [OPTIONS] file|URL|- [REPOSITORY[:TAG]]

Import the contents from a tarball to create a filesystem image

Aliases:
  docker image import, docker import

Options:
  -c, --change list       Apply Dockerfile instruction to the created image
  -m, --message string    Set commit message for imported image
      --platform string   Set platform if server is multi-platform capable
```


`docker import` 命令用于从一个 tar 归档文件、URL 或者标准输入（stdin）导入内容来创建一个新的 Docker 镜像。这个命令通常与 `docker export` 一起使用，以将容器的文件系统导出为 tar 文件，然后在另一台机器上重新导入为镜像。

以下是 `docker import` 的基本用法：


- `-c, --change list`: 允许你应用 Dockerfile 指令到新创建的镜像中。例如，你可以设置工作目录或暴露端口。每个更改应该按照 Dockerfile 指令的格式提供，并且可以指定多个更改。
- `-m, --message string`: 为导入的镜像设置提交信息（commit message）。这可以帮助你记住镜像是如何创建的以及它代表的内容。
- `--platform string`: 如果 Docker 服务器支持多平台，你可以指定要创建的镜像的目标平台（例如 linux/amd64, linux/arm64, windows/amd64 等）。

### 使用示例

1. **从本地 tar 文件导入**:
你可以从一个本地的 tar 文件创建一个新的镜像。例如，如果你有一个名为 `my_container_backup.tar` 的 tar 文件，你可以这样做：

   ```bash
   docker import my_container_backup.tar my_new_image:latest
   ```

2. **从 URL 导入**:
也可以直接从一个 URL 导入 tar 文件。比如，如果 tar 文件托管在一个 HTTP 服务器上，你可以这样操作：

   ```bash
   docker import http://example.com/path/to/my_container_backup.tar my_new_image:latest
   ```

3. **从标准输入导入**:
你可以通过管道从标准输入导入 tar 文件。这在结合其他命令时特别有用，比如当你想解压一个 tar.gz 文件并立即导入它作为新的镜像：

   ```bash
   gunzip -c my_container_backup.tar.gz | docker import - my_new_image:latest
   ```

4. **应用 Dockerfile 指令**:
在导入时，你可以添加一些 Dockerfile 指令来修改新镜像。例如，如果你想设置一个工作目录和暴露一个端口，你可以这样做：

   ```bash
   docker import -c "WORKDIR /app" -c "EXPOSE 8080" my_container_backup.tar my_new_image:latest
   ```

5. **设置提交信息**:
为了记录镜像的来源或创建的目的，你可以添加一个提交信息：

```bash
docker import -m "Imported from a backup of my_container" my_container_backup.tar my_new_image:latest
```

6. **指定平台**:
如果你需要创建一个多平台兼容的镜像，你可以指定目标平台：

```bash
docker import --platform linux/amd64 my_container_backup.tar my_new_image:latest
```

请注意，`docker import` 创建的镜像不会包含原始容器的元数据，如已安装的包管理器的历史记录、环境变量等。如果你需要保留这些信息，你应该考虑使用 `docker commit` 来创建一个新的镜像，或者使用 Dockerfile 来构建镜像。





### **理解 Docker 镜像和容器**
- **镜像 (Image)**：镜像是只读模板，包含了应用程序及其所有依赖项。镜像可以用来创建容器。
- **容器 (Container)**：容器是镜像的一个运行实例。容器是独立的、隔离的运行环境，可以在其中执行应用程序。

## 第二阶段：深入学习

### 5. **编写 Dockerfile**
- **Dockerfile** 是一个文本文件，包含一系列指令，用于定义如何构建 Docker 镜像。
- **常用指令**：
- `FROM`：指定基础镜像。
- `RUN`：在镜像构建过程中执行命令。
- `COPY` 或 `ADD`：将文件或目录复制到镜像中。
- `WORKDIR`：设置工作目录。
- `EXPOSE`：声明容器运行时要监听的端口。
- `CMD` 或 `ENTRYPOINT`：指定容器启动时要执行的命令。
- **示例 Dockerfile**：
```dockerfile
FROM python:3.9-slim
WORKDIR /app
COPY requirements.txt .
RUN pip install -r requirements.txt
COPY . .
CMD ["python", "app.py"]
```

### 6. **构建和推送自定义镜像**
- **构建镜像**：使用 `docker build -t <image_name> .` 命令从 Dockerfile 构建镜像。
- **推送镜像到 Docker Hub**：
- 注册并登录 Docker Hub。
- 使用 `docker tag <image_name> <username>/<repository>:<tag>` 标记镜像。
- 使用 `docker push <username>/<repository>:<tag>` 推送镜像到 Docker Hub。

### 7. **管理容器网络**
- **默认网络**：每个容器都有一个默认的桥接网络（bridge network），允许容器之间通信。
- **自定义网络**：
- 使用 `docker network create <network_name>` 创建自定义网络。
- 使用 `--network <network_name>` 将容器连接到自定义网络。
- **网络模式**：
- `bridge`：默认的隔离网络。
- `host`：容器与主机共享网络命名空间。
- `none`：容器没有网络接口。
- **端口映射**：使用 `-p` 或 `-P` 选项将容器端口映射到主机端口。

### 8. **持久化数据**
- **卷 (Volume)**：卷是 Docker 中用于持久化数据的机制。卷可以独立于容器生命周期存在，并且可以在多个容器之间共享。
- **挂载主机目录**：使用 `-v` 或 `--mount` 选项将主机目录挂载到容器中。
- **命名卷**：使用 `docker volume create <volume_name>` 创建命名卷，并通过 `--mount` 选项将其挂载到容器中。
- **备份和恢复卷**：使用 `docker cp` 命令备份和恢复卷中的数据。

### 第三阶段：高级主题

### 9. **使用 Docker Compose**
- **Docker Compose** 是一个用于定义和运行多容器 Docker 应用程序的工具。它使用 `docker-compose.yml` 文件来定义服务、网络和卷。
- **编写 `docker-compose.yml` 文件**：
```yaml
version: '3'
services:
web:
image: nginx
ports:
- "80:80"
volumes:
- ./html:/usr/share/nginx/html
db:
image: mysql:5.7
environment:
MYSQL_ROOT_PASSWORD: example
networks:
default:
driver: bridge
volumes:
db_data:
```
- **启动和管理多容器应用**：
- `docker-compose up`：启动所有服务。
- `docker-compose down`：停止并删除所有服务。
- `docker-compose ps`：列出所有服务的状态。
- `docker-compose logs`：查看服务的日志。

### 10. **Docker Swarm 和 Kubernetes**
- **Docker Swarm**：Docker 自带的集群管理工具，用于管理多个 Docker 主机上的容器。它允许您创建和管理一个由多个节点组成的 Docker 集群。
- **初始化 Swarm**：`docker swarm init`
- **加入节点**：`docker swarm join --token <token> <manager-ip>:<port>`
- **部署服务**：`docker service create --name <service_name> <image_name>`
- **Kubernetes (K8s)**：Kubernetes 是一个更强大的容器编排平台，广泛用于生产环境中的大规模容器管理。它提供了更多的功能，如自动扩展、负载均衡、滚动更新等。
- **安装 Minikube**：在本地环境中安装和运行 Kubernetes 集群。
- **使用 kubectl**：Kubernetes 的命令行工具，用于管理和操作集群。
- **部署应用程序**：使用 `kubectl apply -f <yaml_file>` 部署应用程序。

### 11. **安全性和最佳实践**
- **最小权限原则**：尽量使用非特权用户运行容器，避免使用 `root` 用户。
- **限制资源使用**：使用 `--memory` 和 `--cpus` 选项限制容器的资源使用，防止其占用过多资源。
- **定期更新镜像**：确保使用最新的镜像版本，以获得最新的安全补丁。
- **使用 Docker Content Trust (DCT)**：启用 DCT 可以确保只拉取经过签名的镜像，增强安全性。
- **日志和监控**：使用日志收集工具（如 ELK Stack）和监控工具（如 Prometheus、Grafana）来跟踪容器的运行状态和性能。

## 第四阶段：项目实战

### 12. **构建一个完整的 Docker 化应用**
- **选择一个项目**：可以选择一个简单的 Web 应用（如 Flask、Node.js）或一个复杂的微服务架构。
- **容器化应用程序**：为每个服务编写 Dockerfile 和 `docker-compose.yml` 文件，确保所有依赖项都正确配置。
- **部署到生产环境**：将应用程序部署到云平台（如 AWS、Google Cloud、Azure）或本地服务器，使用 Docker Swarm 或 Kubernetes 进行编排。
- **持续集成/持续部署 (CI/CD)**：集成 CI/CD 工具（如 Jenkins、GitLab CI、GitHub Actions）来自动化构建、测试和部署流程。


## 网络

容器内部可以直接使用容器ip+容器端口(不是映射之后的端口)进行通信

如果想让docker容器之间网络使用域名进行通讯，还需要创建一个新的网络空间，默认启动的docker是在docker0网络空间，但是docker0网络空间不能进行域名通讯。

```bash
docker network create mynet
# 在多个容器加入同一个网络空间之后，可以直接通过容器名+端口进行访问
docker run -d --name web1 --network mynet nginx
# 可以参考redis主从复制集群的实现
```



## 总结

通过以上步骤，您可以逐步掌握 Docker 的核心概念和高级功能。学习 Docker 不仅可以帮助您更好地理解和使用容器化技术，还可以提高您的开发效率和应用程序的可维护性。如果您有更多具体的问题或需要进一步的帮助，请随时告知！





## 常见命令

.docker & docker hub
![[c8116066bdbf295a7c9fc25b87755dfe.jpg]]

| 命令  | 示例  | 说明  |
| --- | --- | --- |
| docker exec| docker exec -it alpine sh| 对正在运行的容器执行一个命令，效果和docker run命令类似但是不会创建新容器|
| docker ps| docker ps -a| 查看正在运行的容器，-a参数可以查看所有容器|
| docker stop| docker stop alpine| 停止容器，可以强行停止正在运行的容器，支持短键(CONTAINER ID的前三位数字)|
| docker start| docker start alpine| 再次启用已经停止的容器|
| docker rm| docker rm alpine| 彻底删除容器|


## 镜像制作(Dockerfile)

**镜像的完整名字由两个部分组成，名字和标签，中间用 `:` 连接起来

.常用镜像操作命令清单


| 命令| 说明|
|---|---|
| docker create| 创建容器|
| docker pull| 从镜像仓库拉取镜像|
| docker images| 列出当前本地已有的镜像|
| docker rmi| 删除不再使用的镜像|
| docker inspect nginx:alpine| 查看镜像分层信息（元数据）|
| docker attach / exec| 进入容器|
| docker export| 导出容器|
| docker import| 导入容器|


容器镜像内部并不是一个平坦的结构，而是由许多的镜像层组成的，每层都是只读不可修改的一组文件，相同的层可以在镜像之间共享，然后多个层像搭积木一样堆叠起来，再使用一种叫“ **Union FS联合文件系统**”的技术把它们合并在一起，就形成了容器最终看到的文件系统


![[c750a7795ff4787c6639dd42bf0a473f.png]]


*Dockerfile格式*

.Dockerfile
```dockerfile
# Dockerfile.busybox
FROM busybox                  # 选择基础镜像
# 使用CMD指令指定容器启动时默认运行的命令
CMD ["echo", "hello world"]   # 使用JSON数组格式指定命令和参数

# 使用COPY命令将文件从构建上下文复制到镜像中
# COPY <src> <dest>
COPY . /app

# 使用WORKDIR指令设置工作目录
WORKDIR /app

# 使用RUN指令执行构建时的命令
# RUN <command>
RUN apt-get update && apt-get install -y \
    package1 \
    package2

# 使用ENV指令设置环境变量
ENV MY_VAR=my_value

# 使用ARG指令定义构建参数
ARG BUILD_VERSION=1.0
```

.格式说明
****
1. 第一行必须是 `FROM` 指令，用来指定基础镜像
2. `CMD` 它指定 `docker run` 启动容器时默认运行的命令，建议使用JSON数组格式
3. 源码配置等文件可以使用COPY命令打包到镜像中，但是拷贝的源文件必须是构建上下文的路径里面的，不能随意制定文件。
4. 一条指令只能占一行，如果指令太长，可以使用反斜杠 `\` 换行，多个命令之间使用&&符号进行连接
5. RUN后面执行的shell命令可以放到一个单独的文件中进行执行
6.  `ARG` 创建的变量只在镜像构建过程中可见，容器运行时不可见，而 `ENV` 创建的变量不仅能够在构建镜像的过程中使用，在容器运行时也能够以环境变量的形式被应用程序使用
****

##  Docker hub

官方镜像地址：  https://hub.docker.com[https://hub.docker.com]，调用docker pull命令时默认情况下就是从官方镜像里拉去

1. 不指定用户名默认下载官方镜像
2. 如果需要下载制定用户的镜像，需要执行用户名的方式进行下载，如 `bitnami/nginx`

### 镜像命名规则

- 版本 ： 主版本 + 次版本 + 补丁号
- rc: release candidate候选版本
- tags: slim 小，镜像经过精简，fat 包含开发调试工具的版本

##  容器和外界互联互通

### 容器内部和外部之间数据复制

当需要在docker容器内部和外部交互数据时可以使用 `docker cp` 命令。

使用 `docker ps` 命令查看当前运行容器的容器ID，比如这里选一个 `ed1` 的容器，那么将当前路径下的 `a.txt` 移动到docker容器下，只需要执行命令 `docker cp ./a.txt ed1:/tmp/`

查看是否执行成功：

```bash
// 这里的 -i 是保持stdin打开
// t是打开一个tty终端，通常是需要命令行交互时使用该参数
$ docker exec -it ed1 sh
# ls /tmp
a.txt
# exit
```

同样当需要将docker容器里面的内容复制出来时，只需要执行 `docker cp ed1:/tmp/a.txt ./a.txt` 即可，用法和linux命令cp基本一致。

### 共享主机上的文件

复制一两次文件共享还行，经常性的文件来往互通还是要靠文件共享来实现，docker中想实现文件共享非常简单，只需要在启动容器 `docker run` 时添加上 `-v` 命令格式 `宿主机路径:容器内路径`

.eg
```bash
docker run -d --rm -v /tmp:/tmp redis
```

### 实现网络互通

三种模式：null、host和bridge

- null: 最简单的模式，也就是没有网络的模式，只是允许其他网络插件来自定义网络连接

- host: 直接使用宿主机网络，相当于去掉网络隔离，如果使用宿主机网络，需要再启动时使用 `--net=host` 参数。

```bash
docker run -d --rm --net=host nginx:alpine
```

- bridge: 如果不指定模式，则默认使用的是桥接模式，桥接模式下可以使用，`ip` 指定端口的映射关系，类似于真实路由器提供的NAT功能。

```bash
docker run -d --rm -p 6380:6380 redis:6.0.6 redis-server --port 6380
```

部分容器没有提供 ip, ifconfig等命令，可以通过 `docker inspect d46 | grep IPAddress` 命令查看容器的地址信息。


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



## 使用Dockerfile创建镜像

- 基础镜像信息
- 维护着信息
- 镜像操作指令
- 容器启动指令

```bash
# 使用官方的 Python 基础镜像
FROM python:3.10-slim

# 设置工作目录
WORKDIR /app

# 将当前目录下的所有文件复制到容器中的 /app 目录
COPY . .

# 安装 Python 依赖项
RUN pip install --no-cache-dir -r requirements.txt

# 暴露应用程序运行的端口
EXPOSE 5000

# 设置环境变量，避免 Flask 在生产环境中使用开发服务器
ENV FLASK_ENV=production

# 运行 Flask 应用
CMD ["flask", "run", "--host=0.0.0.0"]
```

docker使用虚拟网桥技术实现宿主机和docker容器之间的互联互通。

### 实战

因为平时使用的 go 语言开发，所以使用 go 语言开发一个简单的 web 服务，并使用 Dockerfile 构建镜像。

创建一个main.go文件，并使用 `go mod init` 创建一个go项目，保证本地目录下存在go.mod和go.sum文件，因为只有则才能保证你在任何地方打包的镜像都是使用相同的依赖包。

.main.go
```bash
package main

import (
	"fmt"
	"net/http"
)

// helloWorld 处理函数返回简单的响应。
func helloWorld(w http.ResponseWriter, r *http.Request) {
	fmt.Fprintf(w, "Hello, World!")
}

func main() {
	// 设置路由，当访问根路径"/"时调用helloWorld函数处理请求。
	http.HandleFunc("/", helloWorld)

	// 使用8080端口启动HTTP服务器，并在控制台上打印出启动信息。
	fmt.Println("Starting server at port 8080")
	if err := http.ListenAndServe(":8080", nil); err != nil {
		fmt.Println(err)
	}
}
```

- 创建Dockerfile文件，并添加以下内容：

```bash
# 导入基础镜像golang:alpine
FROM golang:alpine AS builder

# 设置环境变量
ENV GO111MODULE=auto \
    CGO_ENABLED=0 \
    GOOS=linux \
    GOARCH=amd64 \
    GOPROXY="https://goproxy.cn,direct"

# 创建并移动到工作目录（可自定义路径）
WORKDIR /build

# 将代码复制到容器中
COPY . .

# 将代码编译成二进制可执行文件,文件名为 app
RUN go build -o app .

# 利用scratch创建一个小镜像
FROM scratch

# 从builder镜像中把/app 拷贝到当前目录
COPY --from=builder /build/app /

# 声明服务端口
EXPOSE 8080

# 启动容器时运行的命令
CMD ["/app"]
# 构建镜像
# docker build . -t go_app:v1.0.0.0

```

- 执行构建镜像命令：

```bash
docker build . -t go_app:v1.0.0.0
```

image:docker-image.gif[docker-image]

![[image-2025-01-20-21-23-37-300.png]]

## Docker 总结

![[79f8c75e018e0a82eff432786110ef16.jpg]]


## 文章

[IBM Research Report: An Updated Performance Comparison of Virtual Machines and Linux Containers](https://domino.research.ibm.com/library/cyberdig.nsf/papers/0929052195DD819C85257D2300681E7B/%3Czeuschar%3EMY_ZUES_CHAR%3C/zeuschar%3EFile/rc25482.pdf)
[An Introduction to Docker and Analysis of its Performance](http://paper.ijcsns.org/07_book/201703/20170327.pdf)
[Docker Monitoring with the ELK Stack: A Step-by-Step Guide](https://logz.io/learn/docker-monitoring-elk-stack/)
[Eight Docker Development Patterns](https://hokstad.com/docker/patterns) 八个 Docker 的开发模式：共享基础容器、共享同一个卷的多个开发容器、开发工具专用容器、测试环境容器、编译构建容器、防手误的安装容器、默认服务容器、胶黏容器。
[awesome-docker](https://github.com/veggiemonk/awesome-docker)



