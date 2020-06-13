# Docker笔记

### LXC(Linux Container)

LXC(LinuxContainer)是来自于Sourceforge网站上的开源项目，LXC给Linux用户提供了用户空间的工具集，用户可以通过LXC创建和管理容器，在容器中创建运行操作系统就可以有效的隔离多个操作系统，实现操作系统级的虚拟化。

早期docker容器技术就是基于LXC进行构建，即LXC增强版，LXC的二次封装发行版。

Docker 1.8中LXC被deprecated，现在Docker 1.10，LXC彻底出局，取而代之的为libcontainer(runC容器运行时的环境标准)。

### Control Groups(cgroups)

cgroups实现了对资源的配额和度量。

- blkio 块设备
- cpu：CPU
- cpuacct: CPU资源使用报告
- cpuset:多处理器平台上的CPU集合
- devices：设备访问
- freezer: 挂起或恢复任务
- memory：内存用量级别
- perf_event:对cgroups中的任务进行统一性能测试
- net_cls: cgroup中的任务创建的数据报文类别标识符

### 常用虚拟化技术

- 主机级虚拟化：虚拟化整个物理硬件平台
	两种类型：
		I:	直接在硬件上安装虚拟机管理器hypervisor 通过硬件上的虚拟机管理器上安装使用虚拟机， 无底层HOST
		II: 通过在宿主机上VmWare、virtualbox、KMV软件 HOST， 通过HOSTOS上的软件虚拟机软件上创建使用虚拟机
- 容器级虚拟化(Docker:分层构建，联合挂载)

###  Linux Namespaces 6种隔离机制

| namespace | 系统调用参数  | 隔离内容                 | 内核版本 |
|    :-:    |      :-:      |            :-:           |    :-:   |
| UTS       | CLONE_NEWUTS  | 主机名和域名             | 2.6.19   |
| IPC       | CLONE_NEWIPC  | 信号量消息队列及共享内存 | 2.6.19   |
| PID       | CLONE_NEWPID  | 进程编号                 | 2.6.24   |
| Network   | CLONE_NEWNET  | 网络设备、网络栈、端口等 | 2.6.29   |
| Mount     | CLONE_NEWNS   | 挂载点                   | 2.6.19   |
| User      | CLONE_NEWUSER | 用户及用户组             | 3.8      |

### 安装docker

安装docker系统(Linux)要求： 1.x64CPU； 2. Linux Kernel 3.10+; 3. linux cgroups and namespaces

- CentOS：CentOS7 或更高版本满足最低系统要求:

  - 清理旧版
  
    ```shell
    //清理旧版docker
    sudo yum remove docker  docker-client  docker-client-latest \
             docker-common  docker-latest  docker-latest-logrotate \
             docker-logrotate  docker-engine
    ```
    
  - 设置并使用清华大学的仓库源
  
    ```shell
    yum-config-manager --add-repo https://mirrors.tuna.tsinghua.edu.cn/docker-ce/linux/centos/docker-ce.repo
    //vim docker-ce.repo,将repo里的源地址https://download.docker.com/替换为https://mirrors.tuna.tsinghua.edu.cn/docker-ce/
    ```

- Ubuntu

### 版本及环境查看基础命令

- docker -v docker版本信息

  ```shell
  root@iZt4ndfj3stq3mnjve5vzqZ:~# docker -v
  Docker version 18.06.0-ce, build 0ffa825
  ```

- docker version  客户端及服务端详细信息(C/S架构软件)

  ```
  root@iZt4ndfj3stq3mnjve5vzqZ:~# docker version
  Client:
   Version:           18.06.0-ce
   API version:       1.38
   Go version:        go1.10.3
   Git commit:        0ffa825
   Built:             Wed Jul 18 19:11:02 2018
   OS/Arch:           linux/amd64
   Experimental:      false
  
  Server:
   Engine:
    Version:          18.06.0-ce
    API version:      1.38 (minimum version 1.12)
    Go version:       go1.10.3
    Git commit:       0ffa825
    Built:            Wed Jul 18 19:09:05 2018
    OS/Arch:          linux/amd64
    Experimental:     false
  ```

- docker info 详细环境信息

  ```
  root@iZt4ndfj3stq3mnjve5vzqZ:~# docker info
  Containers: 5
   Running: 4
   Paused: 0
   Stopped: 1
  Images: 31
  Server Version: 18.06.0-ce
  Storage Driver: overlay2
   Backing Filesystem: extfs
   Supports d_type: true
   Native Overlay Diff: true
  Logging Driver: json-file
  Cgroup Driver: cgroupfs
  Plugins:
   Volume: local
   Network: bridge host macvlan null overlay
   Log: awslogs fluentd gcplogs gelf journald json-file logentries splunk syslog
  Swarm: inactive
  Runtimes: runc
  Default Runtime: runc
  Init Binary: docker-init
  containerd version: d64c661f1d51c48782c9cec8fda7604785f93587
  runc version: 69663f0bd4b60df09991c08812a60108003fa340
  init version: fec3683
  Security Options:
   apparmor
   seccomp
    Profile: default
  Kernel Version: 4.4.0-117-generic
  Operating System: Ubuntu 16.04.4 LTS
  OSType: linux
  Architecture: x86_64
  CPUs: 1
  Total Memory: 992.1MiB
  Name: iZt4ndfj3stq3mnjve5vzqZ
  ID: ZHUV:5TQW:Q6LU:AIIG:WT7N:LZQF:FTPP:BR5O:FCBX:J4AP:Y2SO:F5G5
  Docker Root Dir: /var/lib/docker
  Debug Mode (client): false
  Debug Mode (server): false
  Registry: https://index.docker.io/v1/
  Labels:
  Experimental: false
  Insecure Registries:
   127.0.0.0/8
  Registry Mirrors:
   https://registry.docker-cn.com/
  Live Restore Enabled: false
  ```

- docker search: 从docker镜像仓库中查找docker镜像

  ```shell
  root@myhost:~# docker search nginx
  ```

### Registry

Registry用于保存docker镜像，包括镜像的层次结构以及元数据，用户可以自建Registry也可以使用官方或第三方的镜像仓库，如docker hub, Registry由2部分组成

- Repository镜像仓库
  - 由某特定的docker镜像的所有迭代版本组成的镜像仓库
  - 一个Registry中可以存在多个Repository
    - Repository分为顶层仓库(official)和用户仓库
    - 用户仓库名称格式为： USER/REPO
  - 每个仓库可以包含多个Tag(标签)，每个标签对应从属于一个镜像，如nginx:stable, nginx:1.14
- Index 
  - 维护用户账号、镜像的校验以及公共命名空间的信息
  - 为Registry提供一个完成用户认证的检索接口， 如： ```docker search nginx```

Docker Registry中镜像仓库由开发人员制作后推送至用户指定的Registry上保存，供其他用户使用

### 镜像

Docker 镜像含有启动系统所需要的文件系统及其内容，因此镜像是用于创建并启动docker容器，镜像是静态的，其采用分层构建，联合挂载机制，最底层为bootfs，其次为rootfs

- bootfs:用于引导系统的文件系统，包括内核(Kernel)以及bootloader,容器启动完成后会被卸载已节约内存资源
- rootfs:位于bootfs之上，表现为docker容器的根文件系统
  - 传统模式中，系统启动时，内核挂载rootfs会先将其挂载为只读模式，系统完整性检查完成后重新将其挂载为读写模式
  - 与传统模式不同的是dockerrootfs始终将其挂为只读模式，而后通过“联合挂载”技术额外挂载一个可写成层，(如：自上而下：writable->(add app image)->(base image)->host->bootfs,自上而下可写)

#### 镜像加速器地址

```shell
sudo vim /etc/docker/daemon.json
```

加入如下JSON配置：

```json
{
	"registry-mirros": ["http://registry.docker-cn.com", "http://xxx.com"]
}
```

保存后重启docker

```powershell
sudo systemctl daemon-reload
sudo systemctl restart docker
```

#### 镜像制作方式

- [Dockerfile](#dockerfile)

- 基于容器制作

  > ```shell
  > #docker commit [OPTIONS] CONTAINER [REPOSITORY[:TAG]]
  > root@iZt4nd:~# docker exec -it ff14b1a4177d /bin/bash
  > ##todo someting in container 
  > ## vi /entrypoint.sh
  > # ....
  > #exit
  > root@iZt4nd:~# docker commit -p -m "nginx config in production" ff14b1a4177d
  > root@iZt4nd:~# docker image ls
  > REPOSITORY          TAG             IMAGE ID            CREATED             SIZE
  > <none>              <none>          21155df3680e        5 seconds ago       952MB
  > root@iZt4nd:~# docker tag 21155df3680e recminy/study-blog:0.1
  > sha256:21155df3680e20b69ef9d3afb44db1609e861ea7905c73cd390ac63d22ae7ec9
  > #查看生成的Image
  > REPOSITORY          TAG             IMAGE ID            CREATED             SIZE
  > recminy/study-blog  0.1             21155df3680e        3 minutes ago       952MB
  > #完整
  > root@iZt4nd:~# docker commit -a "recminy <rewminy@qq.com>" -m "add markdown file support" -c "CMD /entrypoint.sh" -p pma recminy/study-blog:0.2
  > #后续可以使用docker login -u 相关命令来登录dockerhub 把自己的镜像push到dockerhub、阿里云等远端仓库，阿里云相关操作可登录阿里云控制台查看详细文档
  > ```

- dockerhub

#### PUSH 镜像至阿里云

详细操作登录阿里云管理控制台:容器与镜像查看操作指南

#### 镜像导入导出

- 导出 

   使用docker save 将镜像导出为归档文件

  ```shell
  root@iZt4nd:~# docker save -o nginx.tar.gz centos/php:7.1
  #可以同时导出多个镜像
  # docker save -o /path/to/file image1 image2
  ```

- 导入：直接用其他服务器导出的分发镜像快速导入

  ```shell
  root@iZt4nd:~# docker load -i nginx.tar.gz
  ```

#### 镜像常用操作命令：

>     build       Build an image from a Dockerfile
>     history     Show the history of an image
>     import      Import the contents from a tarball to create a filesystem image
>     inspect     Display detailed information on one or more images
>     load        Load an image from a tar archive or STDIN
>     ls          List images
>     prune       Remove unused images
>     pull        Pull an image or a repository from a registry
>     push        Push an image or a repository to a registry
>     rm          Remove one or more images
>     save        Save one or more images to a tar archive (streamed to STDOUT by default)
>     tag         Create a tag TARGET_IMAGE that refers to SOURCE_IMAGE

操作实例：

- docker pull：从镜像仓库中拉取镜像

  ```shell
  #docker [image] pull <registry>[:port]/<namespace/>name<:tag>
  #docker pull quay.io/crio/redis:alpine
  #docker image pull quay.io/blockstack/jenkins:docker
  root@myhost:~# docker pull nginx
  latest: Pulling from library/nginx
  c499e6d256d6: Pull complete 
  74cda408e262: Pull complete 
  ffadbd415ab7: Pull complete
  ```

- docker images: 列出当前系统已拉取镜像

  ```shell
  root@myhost:~# docker images
  REPOSITORY                                        TAG                 IMAGE ID            CREATED             SIZE
  redis                                             latest              4cdbec704e47        30 hours ago        98.2MB
  nginx                                             latest              ed21b7a8aee9        35 hours ago        127MB
  ```



### 容器常用操作命令

启动容器时候，docker daemon会试图从本地获取相关镜像，本地不存在时，会将其从镜像仓库(Registry)中下载该镜像会保存到本地

>     attach      Attach local standard input, output, and error streams to a running container
>     commit      Create a new image from a container's changes
>     cp          Copy files/folders between a container and the local filesystem
>     create      Create a new container
>     diff        Inspect changes to files or directories on a container's filesystem
>     exec        Run a command in a running container
>     export      Export a container's filesystem as a tar archive
>     inspect     Display detailed information on one or more containers
>     kill        Kill one or more running containers
>     logs        Fetch the logs of a container
>     ls          List containers
>     pause       Pause all processes within one or more containers
>     port        List port mappings or a specific mapping for the container
>     prune       Remove all stopped containers
>     rename      Rename a container
>     restart     Restart one or more containers
>     rm          Remove one or more containers
>     run         Run a command in a new container
>     start       Start one or more stopped containers
>     stats       Display a live stream of container(s) resource usage statistics
>     stop        Stop one or more running containers
>     top         Display the running processes of a container
>     unpause     Unpause all processes within one or more containers



早期docker未分组规范命令,如

>docker images 规范写法：docker image ls
>docker pull 规范写法：docker image pull
>docker stop nginx 规范写法：docker container nginx

### Docker 网络

docker提供bridge(默认)、host、none三种网络

```shell
root@iZt4nd:~# docker network ls
NETWORK ID          NAME                DRIVER              SCOPE
99b1ef7cc90d        bridge              bridge              local
ba0bb92e21e7        host                host                local
01243f50c691        none                null                local
9c1e5f866ccf        server_default      bridge              local
```

docker拥有四种网络模型

- Closed container 封闭式

  ```shell
  root@iZt4nd:~# docker run --name b1 -it --network none --rm busybox #只有lo接口
  ```

- Bridged contrainer 

  - 启动容器时默认为桥接式网络

    ```shell
    # 默认--network=bridge
    root@iZt4nd:~# docker run --name t1 -it --network bridge --rm -h "redis.gushanxia.com"
    ```

  - 模拟docker虚拟网卡命令：利用ip命令手动创建虚拟网卡并分配给不同的网络名称空间

    ```shell
    root@iZt4nd:~# ip netns ls
    root@iZt4nd:~# ip netns add r1
    root@iZt4nd:~# root@iZt4nd:~# ip netns add r2
    root@iZt4nd:~# ip netns exec r1 ifconfig -a
    #创建一堆veth网卡
    root@iZt4nd:~# ip link add name veth1.1 type veth peer name veth1.2
    root@iZt4nd:~# ip link show
    #将veth1.2移动到r1网络名称空间
    root@iZt4nd:~# ip link set dev veth1.2 setns r1
    root@iZt4nd:~# ip netns exec r1 ifconfig -a
    #手动将veth1.2改名为eth0
    root@iZt4nd:~# ip netns exec r1 ip link set dev veth1.2 name eth0
    root@iZt4nd:~# ifconfig veth1.1 10.1.0.1/24 up
    root@iZt4nd:~# ip netns exec r1 ifconfig eth0 10.1.0.2/24 up
    ```

- Joined container 联盟式网络，共享IPC、Net、UTS三种名称空间

  联盟式容器是指使用某个已存在容器的网络接口的容器，接口被联盟内的各容器共享使用；故联盟式容器彼此间完全无隔离。

  ```shell
  #创建一个监听于2222端口的httpd服务容器
  root@iZt4nd:~# docker run --name web -d -it -p 2222 busybox:latest /bin/httpd -p 2222 -f
  root@iZt4nd:~# docker run -it --name joined --net container:web busybox netstat -tan
  ```

  联盟式容器彼此虽然共享同一网络名称空间，但其User、Mount、PID还是隔离的。

  联盟式容器彼此间存在网络端口冲突的可能性，因此通常只会在多个容器上的程序要程序loopback接口互通、或对某已存在的容器的网络属性进行监控时才使用此种模式网络模型

- Host container

  ```shell
  root@iZt4nd:~# docker run --name t1 -it -p 8080 --network host --rm -h "gushanxia.com"  nginx
  ```

#### 容器的启动与地址服务暴露(发布)及其他网络特性

- 启动docker容器时候指定网络参数常用参考选项，指定网络，域名，dns，解析域

  ```shell
  # 默认--network=bridge
  root@iZt4nd:~# docker run --name t1 -it --network bridge --rm -h "redis.gushanxia.com" redis 
  root@iZt4nd:~# docker run --name b1 -it --network bridge --rm --hostname "b1.gushanxia.com" busybox 
  root@iZt4nd:~# docker run --name b1 -it --network none --rm busybox #只有lo接口
  #指定dns
  root@iZt4nd:~# docker run --name b1 -it --network bridge --rm -h "busybox.com" --dns '8.8.8.8' --dns "114.114.114.114" busybox
  #注入域名解析至hosts
  root@iZt4nd:~# docker run --name b1 -it --network bridge --rm --add-host "gushanxia.com:0.0.0.0" --add-host "account.gushanxia.com:0.0.0.0" busybox
  ##更多使用docker container run --help
  ```

- 容器启动时候通过-p选项来暴露(发布)容器地址，暴露的四种方式

  - -p containerPort  将指定的容器段楼映射至宿主机所有地址的一个动态端口

    ```shell
root@iZt4nd:~# docker run --name myNginx -p 80 --rm nginx
    #可以通过docker inspect CONTAINER 或者iptables命令查看:~# iptables -t nat -vnL
    root@iZt4nd:~# docker port nyNginx
    root@iZt4nd:~# docker inspect myNginx 查看Ports."80/tcp.HostPort"
    root@iZt4nd:~# iptables -t nat -vnL
    ```
    
  - -p hostPort>:containerPort 容器端口containerPort映射至宿主机端口hostPort

    ```shell
root@iZt4nd:~# docker run --name myWeb -p 8080:80 nginx:latest
    ```
  
  - -p ip::containerPort 指定的容器端口containerPort映射至主机指定的ip的动态端口
  
    ```shell
root@iZt4nd:~# docker run --name myWeb -p 127.0.0.1::80 nginx:latest
    ```
  
  - -p ip:hostPort:containerPort  指定的容器端口containerPort映射至主机指定的ip的端口hostPort

    ```shell
  root@iZt4nd:~# docker run --name myWeb -p 127.0.0.1:8080:80 nginx:latest
    #curl 127.0.0.1:8080
  ```
  
- 自定义docker0桥网络属性: /etc/docker/daemon.json

  >```json
  >{
  >	"bip": "192.168.1.5/24",
  >    "default-gateway": "",
  >	"dns": ["10.20.1.2", "10.20.1.3"]
  >}
  >```

- 自动以docker.sock

  docker守护进程的C/S默认监听Unix socket: `/var/run/docker.sock`,如果使用TCP套接字,则编辑`/etc/docker/daemon.json`

  ```json
  "hosts": ["tcp://0.0.0.0:2375", "unix:///var/run/docker.sock"]
  ```

  也可以直接向dockerd传 -H 连接远程

  ```shell
  root@iZt4nd:~# docker -H 172.20.0.1:2375 ps
  ```


### Docker 存储卷(Volume)

卷是容器上的一个或多个目录，此类目录可绕过联合文件系统，与宿主机上的某目录发生绑定关系，在容器运行期间产生的数据，即使在容器销毁后，只要不删除宿主机的特定目录数据不会丢失。使用Volume有如下优势

- 存储于宿主机系统，便于访问及读写
- 容器间数据共享方便
- 删除容器不会导致数据丢失

docker有两种类型的Volume

- BInd mount volume

  - a volume that points to a user-specified location on the host file system：自行指定容器及宿主机的目录绑定关系

  ```shell
  root@iZt4nd:~# docker run -it -v HOSTDIR:VOLDIR --name bbox busybox
  root@iZt4nd:~# dockerinspect -f {{.Mounts}} b1
  ```

- Docker-managed volume

  - the docker daemon created managed volume in a portion of the host's file sytem that's owned by docker,用户仅需指定容器内的目录，docker daemon会自行管理宿主机上的存储目录

  ```shell
  root@iZt4nd:~# docker run -it --name b1 -v /data/web busybox
  #查看容器的卷、卷标识符及挂载的主机目录
  root@iZt4nd:~# dockerinspect -f {{.Mounts}} b1
  ```

共享存储卷

- 多个容器的卷使用同一个主机目录

  ```shell
  root@iZt4nd:~# docker run -it --name b1 -v /data/web/blog:/data busybox
  root@iZt4nd:~# docker run -it --name b2 -v /data/web/blog:/data busybox
  ```

- 复制使用其他容器的卷，为docker run 命令使用 --volumes-from 选项

  ```shell
  root@iZt4nd:~# docker run -it --name b1 -v /data/web/blog:/data busybox
  root@iZt4nd:~# dcoker run -it --name b2 --volumes-from b1 busybox
  ```

### <a id="dockerfile">Dockerfile</a>

Dockerfile 即用来构建镜像的源码

- 格式：
  - #注释
  - 指令及参数
- 指令不区分大小写，但是约定俗称大写形式
- Dockerfile中的指令是自上而下按顺序执行
- 第一个指令必须为 `FROM` 

#### Dockerfile指令列表

- FROM：用来指定制作当前镜像所用的基础镜像，默认docker会在当前主机上找基准镜像，不存在时候会从registry上拉取

```dockerfile
#FROM <repository>[:<tag>] 或 FROM <repository>@<digest>
FROM centos/php:7.2
```

- MAINTAINER：可选，用于让Dockerfile制作者提供本人的详细信息（废弃，被Label替代）

```dockerfile
MAINTAINER “recminy rewminy@qq.com”
```

- LABEL：让用户为镜像指定各种原始数据信息,LABEL为key-value键值对格式信息

```dockerfile
LABEL user=recminy age=29
```

- COPY：用于从Docker宿主机复制文件至创建的新的镜像文件

```dockerfile
#COPY <src> ... <dest>
#COPY ["<src>", ..., "<dest>"]
#<src>必须是build上下文中的路径，不能是其父目录中的文件
#如果<src>是目录，则其内部的文件或子目录会被递归复制，但<src>自身不会被复制
#如果指定多个<src>，或者在<src>中使用通配符，则<dest>必须是目录，且以/结尾
#如果<dest>事先不存在，它将会被自动创建，这包括其父目录路径
COPY index.html /data/web/html/
COPY yum.repos.d /etc/yum.repos.d/
```

- ADD：类似COPY指令，不同于COPY的是ADD支持使用TAR文件和URL路径

>​	1、如果<src>为URL且<dest>不以/结尾，则<src>指定的文件将被下载并直接命名为<dest>
>​	2、如果<src>是本地系统上的压缩归档文件，它将自动被展开为为目录，其行为类似于``"tar -x"``,但是通过URL下载的tar文件不会被自动展开

- WORKDIR：用于为Dockerfile中所有的RUN、CMD、ENTRYPOINT、COPY和ADD指定设定工作目录，每次只影响从它开始向后的指令

```dockerfile
WORKDIR /usr/local/src
ADD nginx ./ # ./表示为/usr/local/src/
WORKDIR /data/web/html
ADD index.html ./blog/ #/data/web/html/blog/
```

- VOLUME：用于在image中创建一个挂载点目录，以挂载Docker host上的卷或其他容器上的卷，如果挂载点目录路径下此前文件存在，docker run会在卷挂载完成后将这些文件复制到新的挂载卷中

```dockerfile
# VOLUME <mountpoint> 
# VOLUME ["<mountpoint>"]
```

- EXPOSE：用于为容器打开指定要监听的端口以实现外部通信，EXPOSE指令一次可以指定多个端口

```dockerfile
# EXPOSE <port>[/<protocol>][<port>[/<protocol>]] #protocol为tcp或udp，默认tcp
EXPOSE 11211/tcp 11211/udp
```

- ENV

- 编辑完Dockerfile后

```shell
#制作镜像
root@iZt4nd:~# docker build -t repo:tag .
```

  
