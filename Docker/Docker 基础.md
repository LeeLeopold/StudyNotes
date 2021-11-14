# Docker 基础学习

> Docker 官方文档 https://docs.docker.com

## 一、Docker简介



> 1、Docker的概念

Docker 是一个开源的应用容器引擎，让开发者可以打包他们的应用以及依赖包（环境）到一个可移植的镜像中，然后发布到任何流行的 Linux或Windows 机器上，也可以实现虚拟化。



> 2、镜像(images)、容器(container)和仓库(DockerHub)

**镜像:**通过运行镜像启动容器，一个镜像是一个可执行的包，其中包括运行应用程序所需要的所有内容-代码，运行时，库、环境变量和配置文件。

**容器:**容器是镜像文件运行时实例

**仓库:**DockerHub相当与Github等的repository，内容是Images



> 3 、容器与虚拟机的区别

容器与原生主机及其他容器共享内核，轻量化。

相比之下，虚拟机需要运行完整的操作系统，需要更多的资源。





## 二、Docker 安装

> 1、基本步骤

**(1)检查系统要求** Docker 支持 64 位版本 CentOS 7/8，并且要求内核版本不低于 3.10，系统是否安装gcc

```shell
# 1.检查系统版本
[root@localhost home]# uname -r
4.18.0-305.3.1.el8.x86_64

# 2.检查是否安装gcc
[root@localhost home]# gcc -v
使用内建 specs。
COLLECT_GCC=gcc
COLLECT_LTO_WRAPPER=/usr/libexec/gcc/x86_64-redhat-linux/8/lto-wrapper
OFFLOAD_TARGET_NAMES=nvptx-none
OFFLOAD_TARGET_DEFAULT=1
目标：x86_64-redhat-linux
配置为：../configure --enable-bootstrap --enable-languages=c,c++,fortran,lto --prefix=/usr --mandir=/usr/share/man --infodir=/usr/share/info --with-bugurl=http://bugzilla.redhat.com/bugzilla --enable-shared --enable-threads=posix --enable-checking=release --enable-multilib --with-system-zlib --enable-__cxa_atexit --disable-libunwind-exceptions --enable-gnu-unique-object --enable-linker-build-id --with-gcc-major-version-only --with-linker-hash-style=gnu --enable-plugin --enable-initfini-array --with-isl --disable-libmpx --enable-offload-targets=nvptx-none --without-cuda-driver --enable-gnu-indirect-function --enable-cet --with-tune=generic --with-arch_32=x86-64 --build=x86_64-redhat-linux
线程模型：posix
gcc 版本 8.4.1 20200928 (Red Hat 8.4.1-1) (GCC) 

# 3.没有安装执行
[root@localhost home]# yum -y install gcc
[root@localhost home]# yum -y install gcc-c++
```

**(2)卸载旧的docker**

```shell
[root@localhost home]# yum remove docker \
>                    docker-client \
>                    docker-client-latest \
>                    docker-common \
>                    docker-latest \
>                    docker-latest-logrotate \
>                    docker-logrotate \
>                    docker-engine
未找到匹配的参数: docker
未找到匹配的参数: docker-client
未找到匹配的参数: docker-client-latest
未找到匹配的参数: docker-common
未找到匹配的参数: docker-latest
未找到匹配的参数: docker-latest-logrotate
未找到匹配的参数: docker-logrotate
未找到匹配的参数: docker-engine
没有软件包需要移除。
依赖关系解决。
无需任何处理。
完毕！



# ps如果要直接删除docker使用
[root@localhost home]# yum list installed | grep docker
containerd.io.x86_64                               1.4.11-3.1.el8                                @docker-ce-stable
docker-ce.x86_64                                   3:20.10.10-3.el8                              @docker-ce-stable
docker-ce-cli.x86_64                               1:20.10.10-3.el8                              @docker-ce-stable
docker-ce-rootless-extras.x86_64                   20.10.10-3.el8                                @docker-ce-stable
docker-scan-plugin.x86_64                          0.9.0-3.el8                                   @docker-ce-stable

# 依次删除：
[root@localhost home]# yum -y remove docker-ce.x86_64
[root@localhost home]# yum -y remove docker-ce-cli.x86_64
[root@localhost home]# yum -y remove containerd.io.x86_64

# 接着删除docker储存的文件、镜像、容器...，该文件存放在 /var/lib/docker 目录下：
[root@localhost home]# rm -rf /var/lib/docker
```

**(3)安装yum-utils**

```shell
[root@localhost home]# yum install -y yum-utils
上次元数据过期检查：2:33:28 前，执行于 2021年11月14日 星期日 17时04分37秒。
软件包 yum-utils-4.0.18-4.el8.noarch 已安装。
依赖关系解决。
无需任何处理。
完毕！
```

**(4)安装docker仓库**

```shell
# 我们使用国内阿里云的镜像，官网的太慢了
[root@localhost home]# yum-config-manager \
>                     --add-repo \
>                     http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
添加仓库自：http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
```

**(5)更新yum软件包索引**

```shell
[root@localhost home]# yum makecache fast
usage: yum makecache [-c [config file]] [-q] [-v] [--version]
                     [--installroot [path]] [--nodocs] [--noplugins]
                     [--enableplugin [plugin]] [--disableplugin [plugin]]
                     [--releasever RELEASEVER] [--setopt SETOPTS]
                     [--skip-broken] [-h] [--allowerasing] [-b | --nobest]
                     [-C] [-R [minutes]] [-d [debug level]] [--debugsolver]
                     [--showduplicates] [-e ERRORLEVEL] [--obsoletes]
                     [--rpmverbosity [debug level name]] [-y] [--assumeno]
                     [--enablerepo [repo]] [--disablerepo [repo] | --repo
                     [repo]] [--enable | --disable] [-x [package]]
                     [--disableexcludes [repo]] [--repofrompath [repo,path]]
                     [--noautoremove] [--nogpgcheck] [--color COLOR]
                     [--refresh] [-4] [-6] [--destdir DESTDIR]
                     [--downloadonly] [--comment COMMENT] [--bugfix]
                     [--enhancement] [--newpackage] [--security]
                     [--advisory ADVISORY] [--bz BUGZILLA] [--cve CVES]
                     [--sec-severity {Critical,Important,Moderate,Low}]
                     [--forcearch ARCH] [--timer]
yum makecache: error: argument timer: invalid choice: 'fast' (choose from 'timer')

# centos8 使用dnf makecache
[root@localhost home]# dnf makecache
CentOS Linux 8 - AppStream                                                                                6.4 kB/s | 4.3 kB     00:00    
CentOS Linux 8 - BaseOS                                                                                   4.4 kB/s | 3.9 kB     00:00    
CentOS Linux 8 - Extras                                                                                   2.7 kB/s | 1.5 kB     00:00    
Docker CE Stable - x86_64                                                                                  40 kB/s | 3.5 kB     00:00    
元数据缓存已建立。
```

**(6)安装docker-ce Docker社区版**

```shell
[root@localhost home]# yum install docker-ce docker-ce-cli containerd.io
上次元数据过期检查：0:01:42 前，执行于 2021年11月14日 星期日 19时55分40秒。
软件包 docker-ce-3:20.10.10-3.el8.x86_64 已安装。
软件包 docker-ce-cli-1:20.10.10-3.el8.x86_64 已安装。
软件包 containerd.io-1.4.11-3.1.el8.x86_64 已安装。
依赖关系解决。
无需任何处理。
完毕！
```

**(7)启动Docker**

```shell
[root@localhost home]# systemctl start docker
```

**(8)查看docker版本**

```shell
[root@localhost home]# docker version
Client: Docker Engine - Community
 Version:           20.10.10
 API version:       1.41
 Go version:        go1.16.9
 Git commit:        b485636
 Built:             Mon Oct 25 07:42:56 2021
 OS/Arch:           linux/amd64
 Context:           default
 Experimental:      true

Server: Docker Engine - Community
 Engine:
  Version:          20.10.10
  API version:      1.41 (minimum version 1.12)
  Go version:       go1.16.9
  Git commit:       e2f740d
  Built:            Mon Oct 25 07:41:17 2021
  OS/Arch:          linux/amd64
  Experimental:     false
 containerd:
  Version:          1.4.11
  GitCommit:        5b46e404f6b9f661a205e28d59c982d3634148f8
 runc:
  Version:          1.0.2
  GitCommit:        v1.0.2-0-g52b36a2
 docker-init:
  Version:          0.19.0
  GitCommit:        de40ad0
```

**(9)测试hello-world**

```shell
[root@localhost home]# docker run hello-world
Unable to find image 'hello-world:latest' locally			# 没有找到hello-world
latest: Pulling from library/hello-world					# 从仓库中取拉取hello-world
2db29710123e: Pull complete 								# 拉取完成
Digest: sha256:cc15c5b292d8525effc0f89cb299f1804f3a725c8d05e158653a563f15e4f685
Status: Downloaded newer image for hello-world:latest

Hello from Docker!											# 执行成功
This message shows that your installation appears to be working correctly.

To generate this message, Docker took the following steps:
 1. The Docker client contacted the Docker daemon.
 2. The Docker daemon pulled the "hello-world" image from the Docker Hub.
    (amd64)
 3. The Docker daemon created a new container from that image which runs the
    executable that produces the output you are currently reading.
 4. The Docker daemon streamed that output to the Docker client, which sent it
    to your terminal.

To try something more ambitious, you can run an Ubuntu container with:
 $ docker run -it ubuntu bash

Share images, automate workflows, and more with a free Docker ID:
 https://hub.docker.com/

For more examples and ideas, visit:
 https://docs.docker.com/get-started/
 
[root@localhost home]# docker run hello-world				# 再次执行hello-world

Hello from Docker!											# 执行成功
This message shows that your installation appears to be working correctly.

To generate this message, Docker took the following steps:
 1. The Docker client contacted the Docker daemon.
 2. The Docker daemon pulled the "hello-world" image from the Docker Hub.
    (amd64)
 3. The Docker daemon created a new container from that image which runs the
    executable that produces the output you are currently reading.
 4. The Docker daemon streamed that output to the Docker client, which sent it
    to your terminal.

To try something more ambitious, you can run an Ubuntu container with:
 $ docker run -it ubuntu bash

Share images, automate workflows, and more with a free Docker ID:
 https://hub.docker.com/

For more examples and ideas, visit:
 https://docs.docker.com/get-started/
```

**(10)查看下载的镜像**

```shell
[root@localhost home]# docker images
REPOSITORY    TAG       IMAGE ID       CREATED       SIZE
hello-world   latest    feb5d9fea6a5   7 weeks ago   13.3kB
centos        latest    5d0da3dc9764   8 weeks ago   231MB
```



> 2配置国内镜像加速

```shell
# 创建目录
[root@localhost home]# sudo mkdir -p /etc/docker
# 修改daemon配置文件/etc/docker/daemon.json来使用加速器
[root@localhost docker]# sudo tee /etc/docker/daemon.json <<-'EOF'
{
  "registry-mirrors": ["https://hybs7fub.mirror.aliyuncs.com"]
}
EOF
{
  "registry-mirrors": ["https://hybs7fub.mirror.aliyuncs.com"]
}
# 重载deamon
[root@localhost docker]# systemctl daemon-reload
# 重启docker
[root@localhost docker]# systemctl restart docker
```

