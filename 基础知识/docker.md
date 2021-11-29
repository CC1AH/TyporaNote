[TOC]





### 零. 体验教程.你可以跳过这部分

安装：https://yeasy.gitbook.io/docker_practice/install/windows

windows10-转移空间：https://stackoverflow.com/questions/62441307/how-can-i-change-the-location-of-docker-images-when-using-docker-desktop-on-wsl2

```
#bash file
#stop docker services, use 
>wsl --list -v
#to check it.
>wsl --export docker-desktop-data "E:\Docker\wsl\data\docker-desktop-data.tar"
>wsl --unregister docker-desktop-data
>wsl --import docker-desktop-data "E:\Docker\wsl\data" "E:\Docker\wsl\data\docker-desktop-data.tar" --version 2
```

官方的引导实验：

```
#bash file
#better running in an empty dir
>docker run --name repo alpine/git clone https:github.com/docker/getting-started.git
>docker cp repo:/git/getting-started/ .
>cd getting-started
>docker build -t tutotial1
>docker run -d -p 5000:80 --name docker-tutorial tutorial1
>docker tag tutorial1 cc1ah/tutorial1
>docker push cc1ah/tutorial1
```



我们从这里manual正式开始：https://yeasy.gitbook.io/docker_practice/introduction/what

### 一. 基本概念

1. docker

   操作系统层面的虚拟化技术（而虚拟机是硬件层面的），提供除了内核外的完整运行时环境。

2. 相比虚拟机的优势

   秒级启动，MB的硬盘使用、接近原生的性能、单机支持上千个容器

3. docker镜像

   一个**特殊的文件系统**，提供容器运行时的程序、库、配置资源和参数，不包括动态数据。在实践中，它被利用Union FS设计成分层存储的架构。因此，它只是一个虚拟的概念，用于表示一组文件簇

4. docker容器

   指**镜像运行时的实体**，实质是拥有独立命名空间（自己的root文件系统、网络配置、进程空间）的进程。容器内的进程运行在隔离的环境中，保证了安全性。

5. docker容器存储层

   容器运行时，是以镜像为基础层，在其上创建一个当前容器的存储层，生命周期和容器一致。容器**不会向存储层中写入数据，而是将数据写入数据卷**，这种设计会将读写直接反应在宿主(或网络存储)，保证性能和稳定性

6. docker仓库

   **仓库是用于存储构建完成镜像的区域。**每个仓库可以包含多个标签，一个标签对应一个镜像。其名字一般以两段式出现。比如‘cc1ah/nginx-proxy’，前者往往意味着 Docker Registry(一个仓库构建工具) 多用户环境下的用户名，后者则往往是对应的软件名。

7. docker registry

   一个仓库提供的服务，包括公开docker registry(开放给用户使用、允许用户管理镜像的 Registry 服务。一般这类公开服务允许用户免费上传、下载公开的镜像，并可能提供收费服务供用户管理私有镜像，比如默认下载源的[docker hub](https://hub.docker.com/))。

   还包括私有dr，提供用户在本地搭建的选项。我们[后面](https://yeasy.gitbook.io/docker_practice/repository/registry)会涉及。

8. docker数据卷（volume）

   可供一个或多个容器使用的特殊目录、可以绕过UFS的存储架构进行容器间数据共享、重用；同时保证修改的即时性和持久性。



### 二.使用镜像

 1. 获取镜像

    镜像是由多层存储所构成。下载也是一层层的去下载，并非单一文件。下载过程中给出了每一层的 ID 的前 12 位。并且下载结束后，给出该镜像完整的 [`sha256`](https://zhuanlan.zhihu.com/p/94619052) 的摘要，符合[验证结果](https://hash.online-convert.com/sha256-generator)，以确保下载一致性

    ```
     # 语法：docker pull [选项] [Docker Registry 地址[:端口号]/]仓库名[:标签]
     > docker pull -q docker.io/library/ubuntu:18.04
     # 默认的docker.io（dockerhub）和library(仓库用户名段)都可以省略。
     
     # 帮助以查看选项内容：
     > docker pull --help
     
     # 运行镜像
     > docker run -it --rm ubuntu:18.04 bash
    ```

    关于运行：

    `-it`：这是两个参数， `-i`：交互式操作，一个是 `-t` 终端。这里打算进入 `bash` 执行命令并查看返回结果，因此我们需要交互式终端。

    `--rm`：容器退出后随之将其删除。默认情况下，为了排障需求，退出的容器并不会立即删除，除非手动 `docker rm`。我们这里只是随便执行个命令，看看结果，不需要排障和保留结果，因此使用 `--rm` 可以避免浪费空间。

    `ubuntu:18.04`：这是指用 `ubuntu:18.04` 镜像为基础来启动容器。

    `bash`：放在镜像名后的是 **命令**，这里我们希望有个交互式 Shell，因此用的是 `bash`，比如我们可以接着执行 cat /etc/os-release查看容器内系统的版本(显然是ubuntu18.04的对应版本)。

	2. 列出镜像

    ```
    >docker image ls #结果包括仓库名、标签、镜像 ID、创建时间 以及 所占用的空间(比docker hub上大，因为远程是压缩过的)。
    >docker image ls ubuntu
    >docker image ls ubuntu:18.04 # 其他格氏和条件请参考help或者manual
    ```

    注意总体积和不一定是实际存储消耗，因为镜像的分层存储，一些镜像可能会使用相同的层。我们使用

    ```
    >docker system df
    ```

    查看镜像、容器和数据卷所占的空间。

    在镜像列表中，我们看到仓库名和标签均为<none>的**虚悬镜像**，它属于历史问题，没有任何价值和作用。但是仓库名非空，标签为<none>的镜像称为**中间层镜像**，它可能是其他镜像的重要依赖。

	3. 删除镜像

    ```
    #  docker image rm [选项] <镜像1> [<镜像2> ...]
    # <镜像> 可以是 镜像短 ID、镜像长 ID、镜像名 或者 镜像摘要, 我们推荐使用镜像摘要精确删除镜像（但是其实desktop里面点点点更加常用...）
    >docker image rm node@sha256:b4f0e0bdeb578043c1ea6862f0d40cc4afe32a4a582f3be235a3b164422be228
    output: Untagged: node@sha256:b4f0e0bdeb578043c1ea6862f0d40cc4afe32a4a582f3be235a3b164422be228
    ```

    删除行为分为两类，一类是 `Untagged`，另一类是 `Deleted`。我们之前介绍过，镜像的唯一标识是其 ID 和摘要，而一个镜像可以有多个标签。

    因此**当我们使用命令删除镜像的时候，实际上是在要求删除某个标签的镜像**。所以首先需要做的是将满足我们要求的所有镜像标签都取消，这就是我们看到的 `Untagged` 的信息。因为一个镜像可以对应多个标签，因此**当我们删除了所指定的标签后，可能还有别的标签指向了这个镜像**，如果是这种情况，那么 `Delete` 行为就不会发生。所以并非所有的 `docker image rm` 都会产生删除镜像的行为，有可能仅仅是取消了某个标签而已。

    镜像是多层存储结构，删除是从上层向基础层方向依次进行判断删除。**很有可能某个其它镜像正依赖于当前镜像的某一层**。这种情况依旧不会触发删除该层的行为。

    还需要注意的是**容器对镜像的依赖**。同样不可以删除这个镜像（容器是以镜像为基础，再加一层容器存储层，组成这样的多层存储结构去运行）

    

### 三. 操作容器

 1. docker run 命令

    ```
    # 基于镜像新建一个容器并启动
    #  docker run [OPTIONS] IMAGE [COMMAND] [ARG...]
    >docker run -t -i ubuntu:18.04 /bin/bash
    # -t 选项让Docker分配一个伪终端（pseudo-tty）并绑定到容器的标准输入上， -i 则让容器的标准输入保持打开，运行这一行之后ubuntu的终端被打开。
    ```

	2. 可以利用 `docker container start` 命令，直接将一个已经终止（`exited`）的容器启动运行

	3. 守护态运行

    更多的时候，需要让 Docker 在后台运行而不是直接把执行命令的结果输出在当前宿主机下。此时，可以通过添加 `-d` 参数来实现。

    ```
    > docker run -d ubuntu:18.04 /bin/sh -c "while true; do echo hello world; sleep 1; done"
    # 该命令输出结果可以用 docker logs 查看
    # 使用 -d 参数后需要进入容器进行操作，可以使用docker attach或docker exec（推荐）
    > docker run -dit ubuntu
    69d137adef7a8a689cbcb059e94da5489d3cddd240ff675c640c8d96e84fe1f6
    > docker container ls
    ...
    > docker exec -it 69d1 bash
    root@69d137adef7a:/#
    ```

	4. 终止和删除

    ```
    # 查看终止状态容器
    > docker container ls -a
    # 终止运行容器，或者关闭终端/ctrl + c
    > docker container stop 243c 
    #这里用了短id
    
    #删除
    > docker container rm containerName(这个用--name指定，看manual)
    ```

	5. 容器导出到本地文件和从本地文件导入

    ```
    > docker export 7691a814370e > ubuntu.tar
    > cat ubuntu.tar | docker import - test/ubuntu:v1.0
    ```

    注：也可以使用docker load来导入镜像存储文件到本地镜像库，区别在于后者将丢弃所有的历史记录和元数据信息（即仅保存容器当时的快照状态）、此外可以重新指定标签等元数据信息。

    

### 四. 访问仓库

 1. 我们可以把自己的镜像推送到docker  hub

    ```
    > docker tag ubuntu:18.04(镜像名) cc1ah/ubuntu:18.04（本地仓库两段目录）
    > docker push cc1ah/ubuntu:18.04（远程目录）
    > docker search cc1ah
    ```

	2. 我们可以使用search pull拉取docker hub的镜像

	3. 我们可以通过配置docker registry实现本地私有仓库的管理：https://yeasy.gitbook.io/docker_practice/repository/registry

    当然，你可以使用[nexus](http://wangzhangtao.com/2020/06/16/%E5%B8%B8%E7%94%A8%E8%BD%AF%E4%BB%B6-nexus%E7%A7%81%E6%9C%8D/#Nexus%E4%BB%8B%E7%BB%8D)



### 五. 管理数据 待做

### 六. Docker 构建和 Dockerfile  待做 

### 七. Docker Compose 待做

### 八. Kubernetes 待做

### 九. VSCode Docker实战案例 待做