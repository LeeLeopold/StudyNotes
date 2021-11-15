

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

![img](https://gimg2.baidu.com/image_search/src=http%3A%2F%2Fwww.pianshen.com%2Fimages%2F311%2Fbe84854eacb6d9d844f6f6147fa16b97.JPEG&refer=http%3A%2F%2Fwww.pianshen.com&app=2002&size=f9999,10000&q=a80&n=0&g=0n&fmt=jpeg?sec=1639550657&t=088563cafb73ff82a6857a63119015bd)

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



## 三、Docker的基本命令



![img](https://www.mscto.com/wp-content/uploads/2020/02/20200216170310480.png)



> **1、帮助命令**

命令：`Docker xx命令 --help`

```shell
[root@localhost ~]# docker images --help

Usage:  docker images [OPTIONS] [REPOSITORY[:TAG]]

List images

Options:
  -a, --all             Show all images (default hides intermediate images)
      --digests         Show digests
  -f, --filter filter   Filter output based on conditions provided
      --format string   Pretty-print images using a Go template
      --no-trunc        Don't truncate output
  -q, --quiet           Only show image IDs
```

> **2、镜像命令**

**(1) 查看所有镜像**

```shell
docker images
docker images -a 展示所有镜像 
docker images -q 只展示镜像的ID
```



```shell
[root@localhost ~]# docker images
REPOSITORY    TAG       IMAGE ID       CREATED        SIZE
hello-world   latest    feb5d9fea6a5   7 weeks ago    13.3kB
centos        latest    5d0da3dc9764   2 months ago   231MB
```

| 列名       | 含义           |
| ---------- | -------------- |
| REPOSITORY | 镜像的仓库源   |
| TAG        | 镜像的标签     |
| IMAGE ID   | 镜像的ID       |
| CREATED    | 镜像的创建时间 |
| SIZE       | 镜像的大小     |



**(2) 搜索镜像**

```shell
docker search 镜像
docker search 镜像 --filter=starts=3000
```

```shell
# 直接搜索(默认搜索25条)
[root@localhost ~]# docker search mysql
NAME                              DESCRIPTION                                     STARS     OFFICIAL   AUTOMATED
mysql                             MySQL is a widely used, open-source relation…   11682     [OK]       
mariadb                           MariaDB Server is a high performing open sou…   4451      [OK]       
mysql/mysql-server                Optimized MySQL Server Docker images. Create…   868                  [OK]
percona                           Percona Server is a fork of the MySQL relati…   561       [OK]       
phpmyadmin                        phpMyAdmin - A web interface for MySQL and M…   370       [OK]       
centos/mysql-57-centos7           MySQL 5.7 SQL database server                   91                   
mysql/mysql-cluster               Experimental MySQL Cluster Docker images. Cr…   89                   
centurylink/mysql                 Image containing mysql. Optimized to be link…   59                   [OK]
databack/mysql-backup             Back up mysql databases to... anywhere!         52                   
prom/mysqld-exporter                                                              43                   [OK]
deitch/mysql-backup               REPLACED! Please use http://hub.docker.com/r…   41                   [OK]
tutum/mysql                       Base docker image to run a MySQL database se…   35                   
linuxserver/mysql                 A Mysql container, brought to you by LinuxSe…   34                   
schickling/mysql-backup-s3        Backup MySQL to S3 (supports periodic backup…   31                   [OK]
mysql/mysql-router                MySQL Router provides transparent routing be…   23                   
centos/mysql-56-centos7           MySQL 5.6 SQL database server                   20                   
arey/mysql-client                 Run a MySQL client from a docker container      19                   [OK]
fradelg/mysql-cron-backup         MySQL/MariaDB database backup using cron tas…   16                   [OK]
openshift/mysql-55-centos7        DEPRECATED: A Centos7 based MySQL v5.5 image…   6                    
ansibleplaybookbundle/mysql-apb   An APB which deploys RHSCL MySQL                3                    [OK]
devilbox/mysql                    Retagged MySQL, MariaDB and PerconaDB offici…   3                    
idoall/mysql                      MySQL is a widely used, open-source relation…   3                    [OK]
jelastic/mysql                    An image of the MySQL database server mainta…   2                    
widdpim/mysql-client              Dockerized MySQL Client (5.7) including Curl…   1                    [OK]
centos/mysql-80-centos7           MySQL 8.0 SQL database server                   1                    
# 搜索stars数在3000以上的image
[root@localhost ~]# docker search mysql --filter=STARS=3000
NAME      DESCRIPTION                                     STARS     OFFICIAL   AUTOMATED
mysql     MySQL is a widely used, open-source relation…   11682     [OK]       
mariadb   MariaDB Server is a high performing open sou…   4451      [OK] 
```



**(3) 下载镜像**

```shell
docker pull 镜像 ：默认下载最新版本（latest）
docker pull 镜像:版本 ：下载指定的版本
```



```shell
# 下载 最新版本 mysql
[root@localhost ~]# docker pull mysql
Using default tag: latest				# 版本
latest: Pulling from library/mysql		# 分层下载
b380bbd43752: Pull complete 
f23cbf2ecc5d: Pull complete 
30cfc6c29c0a: Pull complete 
b38609286cbe: Pull complete 
8211d9e66cd6: Pull complete 
2313f9eeca4a: Pull complete 
7eb487d00da0: Pull complete 
4d7421c8152e: Pull complete 
77f3d8811a28: Pull complete 
cce755338cba: Pull complete 
69b753046b9f: Pull complete 
b2e64b0ab53c: Pull complete 
# 签名
Digest: sha256:6d7d4524463fe6e2b893ffc2b89543c81dec7ef82fb2020a1b27606666464d87
Status: Downloaded newer image for mysql:latest
docker.io/library/mysql:latest			# 镜像下载地址

# 下载指定版本 mysql, 注意分层下载
[root@localhost ~]# docker pull mysql:5.7
5.7: Pulling from library/mysql
b380bbd43752: Already exists 
f23cbf2ecc5d: Already exists 
30cfc6c29c0a: Already exists 
b38609286cbe: Already exists 
8211d9e66cd6: Already exists 
2313f9eeca4a: Already exists 
7eb487d00da0: Already exists 
a71aacf913e7: Pull complete 
393153c555df: Pull complete 
06628e2290d7: Pull complete 
ff2ab8dac9ac: Pull complete 
Digest: sha256:2db8bfd2656b51ded5d938abcded8d32ec6181a9eae8dfc7ddf87a656ef97e97
Status: Downloaded newer image for mysql:5.7
docker.io/library/mysql:5.7

# 最终镜像
[root@localhost ~]# docker images
REPOSITORY    TAG       IMAGE ID       CREATED        SIZE
mysql         5.7       938b57d64674   3 weeks ago    448MB
mysql         latest    ecac195d15af   3 weeks ago    516MB
hello-world   latest    feb5d9fea6a5   7 weeks ago    13.3kB
centos        latest    5d0da3dc9764   2 months ago   231MB

```

此外 

```shell
docker pull mysql:5.7 <==> docker pull docker.io/library/mysql:5.7
```



**(4) 删除镜像**

```shell
docker rmi -f 镜像ID
```

```shell
# 删除前的镜像
[root@localhost ~]# docker images
REPOSITORY    TAG       IMAGE ID       CREATED        SIZE
mysql         5.7       938b57d64674   3 weeks ago    448MB
mysql         latest    ecac195d15af   3 weeks ago    516MB
hello-world   latest    feb5d9fea6a5   7 weeks ago    13.3kB
centos        latest    5d0da3dc9764   2 months ago   231MB

# 通过ID进行删除
[root@localhost ~]# docker rmi -f 938b57d64674
Untagged: mysql:5.7
Untagged: mysql@sha256:2db8bfd2656b51ded5d938abcded8d32ec6181a9eae8dfc7ddf87a656ef97e97
Deleted: sha256:938b57d64674c4a123bf8bed384e5e057be77db934303b3023d9be331398b761
Deleted: sha256:d81fc74bcfc422d67d8507aa0688160bc4ca6515e0a1c8edcdb54f89a0376ff1
Deleted: sha256:a6a530ba6d8591630a1325b53ef2404b8ab593a0775441b716ac4175c14463e6
Deleted: sha256:2a503984330e2cec317bc2ef793f5d4d7b3fd8d50009a4f673026c3195460200
Deleted: sha256:e2a4585c625da1cf4909cdf89b8433dd89ed5c90ebdb3a979d068b161513de90

# 确认已删除
[root@localhost ~]# docker images
REPOSITORY    TAG       IMAGE ID       CREATED        SIZE
mysql         latest    ecac195d15af   3 weeks ago    516MB
hello-world   latest    feb5d9fea6a5   7 weeks ago    13.3kB
centos        latest    5d0da3dc9764   2 months ago   231MB
```



删除所有的镜像

```shell
# docker images -aq 显示所有的images的id
[root@localhost ~]# docker rmi -f $(docker images -aq)
Untagged: mysql:latest
Untagged: mysql@sha256:6d7d4524463fe6e2b893ffc2b89543c81dec7ef82fb2020a1b27606666464d87
Deleted: sha256:ecac195d15afac2335de52fd7a0e34202fe582731963d31830f1b97700bf9509
Deleted: sha256:451fe04d80b84c0b7aca0f0bbdaa5de7c7ac85a65389ed5d3ed492f63ac092e2
Deleted: sha256:814cbf8bc7f6bb85685e5b803e16a76406c30d1960c566eee76303ffac600600
Deleted: sha256:735f72e1d1b936bb641b6a1283e4e60bf10a0c36f8244a5e3f8c7d58fa95b98a
Deleted: sha256:f2d209a30c3950fadffb2d82e1faa434da0753bee7aacad9cdec7d8a7a83df37
Deleted: sha256:03b9f8c5331d9534d2372a144bcffc8402e5f7972c9e4b85c634bef203ec6d20
Deleted: sha256:80f5487a88b8061855e99782979ed6069a8dd1c7dfbb1eb63fe42a4a9d119436
Deleted: sha256:f791a6c727931d41c51f8bf24ee32a4dbf0169f372b174f1ff89b4836b97c48e
Deleted: sha256:4c88df098412e11a98936509f3cede57f87154b350b0f75d96713f6e1dd56101
Deleted: sha256:fdba3a2cd286d9a5f65fc00f5254048855ae7dc00f3b3fa3356981eb9a7fe6d0
Deleted: sha256:8b3a69042e0da82429d28be0c474e73290ba4908730de22b2200a7aac9b245bd
Deleted: sha256:90afe56a0643f5bf1b1e8ee147b40a8e12b3fdd7e26bc2d2c50180d68dd524d0
Deleted: sha256:e81bff2725dbc0bf2003db10272fef362e882eb96353055778a66cda430cf81b
Untagged: hello-world:latest
Untagged: hello-world@sha256:cc15c5b292d8525effc0f89cb299f1804f3a725c8d05e158653a563f15e4f685
Deleted: sha256:feb5d9fea6a5e9606aa995e879d862b825965ba48de054caab5ef356dc6b3412
Untagged: centos:latest
Untagged: centos@sha256:a27fd8080b517143cbbbab9dfb7c8571c40d67d534bbdee55bd6c473f432b177
Deleted: sha256:5d0da3dc976460b72c77d94c8a1ad043720b0416bfc16c52c45d4847e53fadb6

# 确认已删除
[root@localhost ~]# docker images
REPOSITORY   TAG       IMAGE ID   CREATED   SIZE
```



> **3、容器命令**

首先创建一个centos镜像

```shell
[root@localhost ~]# docker pull centos
Using default tag: latest
latest: Pulling from library/centos
a1d0c7532777: Already exists 
Digest: sha256:a27fd8080b517143cbbbab9dfb7c8571c40d67d534bbdee55bd6c473f432b177
Status: Downloaded newer image for centos:latest
docker.io/library/centos:latest

[root@localhost ~]# docker images
REPOSITORY   TAG       IMAGE ID       CREATED        SIZE
centos       latest    5d0da3dc9764   2 months ago   231MB
```

**(1) 新建容器并启动**

```shell
# 先查看run命令
docker run [可选参数] 镜像
可选参数：
--name="名称"       # 设置容器名
-d                 # 后台方式运行
-it                # 使用交互方式运行，可以进入容器查看内容
-p                 #(小p),指定容器端口
    -p ip:主机端口:容器端口
    -p 主机端口:容器端口（常用）
    -p 容器端口
-P                 #(大P),随机指定容器端口
```

```shell
# 新建并以交互的形式启动容器
[root@localhost ~]# docker run -it centos /bin/bash
[root@e9c5faaeb98e /]# ls
bin  etc   lib	  lost+found  mnt  proc  run   srv  tmp  var
dev  home  lib64  media       opt  root  sbin  sys  usr

# 退出并停止容器
[root@e9c5faaeb98e /]# exit
exit

# 无正在运行的容器
[root@localhost ~]# docker ps
CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES

# 查看所有的容器
[root@localhost ~]# docker ps -a
CONTAINER ID   IMAGE          COMMAND       CREATED         STATUS                          PORTS     NAMES
e9c5faaeb98e   centos         "/bin/bash"   2 minutes ago   Exited (0) About a minute ago             eager_poitras
eaf9789d6e54   feb5d9fea6a5   "/hello"      21 hours ago    Exited (0) 21 hours ago                   recursing_wu
630cd703f20a   feb5d9fea6a5   "/hello"      21 hours ago    Exited (0) 21 hours ago                   optimistic_austin
e3bc4a07a7d5   centos         "/bin/bash"   22 hours ago    Exited (0) 22 hours ago                   upbeat_swartz

# 删除所有的容器
[root@localhost ~]# docker rm -f $(docker ps -aq)
e9c5faaeb98e
eaf9789d6e54
630cd703f20a
e3bc4a07a7d5

# 退出不停止容器（Ctrl + P + Q）
[root@localhost ~]# docker run -it centos /bin/bash 
[root@88493ef7a1a1 /]# [root@localhost ~]# 

# 查看正在运行的容器
[root@88493ef7a1a1 /]# [root@localhost ~]# docker ps
CONTAINER ID   IMAGE     COMMAND       CREATED          STATUS          PORTS     NAMES
88493ef7a1a1   centos    "/bin/bash"   37 seconds ago   Up 36 seconds             nervous_austin

# 再次进入交互模式的容器
[root@localhost ~]# docker attach 88493ef7a1a1
[root@88493ef7a1a1 /]# 

[root@88493ef7a1a1 /]# ls
bin  etc   lib	  lost+found  mnt  proc  run   srv  tmp  var
dev  home  lib64  media       opt  root  sbin  sys  usr
[root@88493ef7a1a1 /]# 

# 退出并关闭容器
[root@88493ef7a1a1 /]# exit
exit
[root@88493ef7a1a1 /]# 
退出并关闭容器

[root@localhost ~]# docker ps
CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES

```



**(2) 列出运行中的容器**

```shell
docker ps 			#查看正在运行的容器
docker ps -a		#查看运行容器的历史记录
docker ps -a -n=2	#查看最近运行的两个容器
docker ps -aq		#查看所有容器的ID
```

```shell
[root@localhost ~]# docker ps -a
CONTAINER ID   IMAGE     COMMAND       CREATED         STATUS                     PORTS     NAMES
88493ef7a1a1   centos    "/bin/bash"   5 minutes ago   Exited (0) 2 minutes ago             nervous_austin
```



**(3) 启动并停止容器**

```shell
docker start 容器ID		#启动容器
docker restart 容器ID		#重启容器
docker stop 容器ID		#停止容器
docker kill 容器ID		#杀死容器
```



**(4) 删除容器**

```shell
docker rm 容器ID 					#删除指定容器（不能删除正在运行的容器）
docker rm -f 容器ID				#强制删除指定容器
docker rm -f $(docker ps -aq)	 #删除所有的容器， -aq：列出所有容器的ID
```



> **4、其他命令**

**(1) 后台启动容器**

```shell
docker run -d centos # 后台启动容器

# 问题：docker ps 后发现并没有centos的进程

# 原因：docker容器使用后台进程，在容器中并没有任何应用（进程），所以是UN呢个通过docker ps 查看到docker的进程的。但是你是可以通过 docker ps -a 进行查看的

# 解决：自定义一小段shell脚本，让容器中有东西在运行
```

```shell
# 查看容器
[root@localhost ~]# docker ps -a
CONTAINER ID   IMAGE     COMMAND       CREATED         STATUS                     PORTS     NAMES
88493ef7a1a1   centos    "/bin/bash"   5 minutes ago   Exited (0) 2 minutes ago             nervous_austin

# 停止容器
[root@localhost ~]# docker stop 88493ef7a1a1
88493ef7a1a1

# 删除容器
[root@localhost ~]# docker rm 88493ef7a1a1
88493ef7a1a1

[root@localhost ~]# docker ps -a
CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES

# 重新启动一个容器（centos），编写一段shell脚本，使得centos中有东西在运行
[root@localhost ~]# docker run -d centos /bin/bash -c "while true; do echo docker is running~;sleep 5;done"
a89fb68af85ba9ee64f50076f84530a58bb0477adb8b717f6b3f620e59607c39

# 容器已启动,且未关闭
[root@localhost ~]# docker ps
CONTAINER ID   IMAGE     COMMAND                  CREATED              STATUS              PORTS     NAMES
a89fb68af85b   centos    "/bin/bash -c 'while…"   About a minute ago   Up About a minute             blissful_brown
```



**(2) 查询日志**

```shell
docker logs -tf --tail number 容器ID
-tf： 			#显示日志
--tail number 	# 显示日志尾部的几条日志
```

```shell
[root@localhost ~]# docker logs -tf --tail 10 a89fb68af85b
2021-11-15T09:18:35.946907100Z docker is running~
2021-11-15T09:18:40.952464700Z docker is running~
2021-11-15T09:18:45.954631200Z docker is running~
2021-11-15T09:18:50.956232100Z docker is running~
2021-11-15T09:18:55.957855700Z docker is running~
2021-11-15T09:19:00.959506200Z docker is running~
2021-11-15T09:19:05.963163700Z docker is running~
2021-11-15T09:19:10.964294600Z docker is running~
2021-11-15T09:19:15.965949100Z docker is running~
2021-11-15T09:19:20.968865000Z docker is running~
2021-11-15T09:19:25.970380300Z docker is running~
```



**(3) 查看docker容器内部的进程信息**

```shell
[root@localhost ~]# docker top a89fb68af85b
UID                 PID                 PPID                C                   STIME               TTY                 TIME                CMD
root                100632              100614              0                   17:15               ?                   00:00:00            /bin/bash -c while true; do echo docker is running~;sleep 5;done
root                100798              100632              0                   17:20               ?                   00:00:00            /usr/bin/coreutils --coreutils-prog-shebang=sleep /usr/bin/sleep 5
```



**(4) 查看容器中的元数据**

```shell
[root@localhost ~]# docker inspect a89fb68af85b
```



**(5) 进入当前正在运行的容器**

```shell
# 方式1,开启一个新的终端
docker exec -it 容器ID bashShell

# 方式2,进入当前正在执行的终端
docker attach 容器ID
```

```shell
# 方式1
[root@localhost ~]# docker exec -it a89fb68af85b /bin/bash
[root@a89fb68af85b /]# 

# 方式2
[root@localhost ~]# docker attach a89fb68af85b
docker is running~
docker is running~

```



**(6) 拷贝容器内的文件到本地**

```shell
# 进入容器，并新建一个文件
[root@localhost ~]# docker exec -it a89fb68af85b /bin/bash
[root@a89fb68af85b /]# cd /home          
[root@a89fb68af85b home]# touch test.txt
[root@a89fb68af85b home]# ls
test.txt

# 退出并关闭容器
[root@a89fb68af85b home]# exit
exit
[root@localhost ~]# docker stop a89fb68af85b
a89fb68af85b

# 并拷贝容器内的文件到本地
[root@localhost ~]# docker cp a89fb68af85b:/home/test.txt /home
[root@localhost ~]# cd /home
[root@localhost home]# ll
总用量 4676
-rw-r--r--.  1 root    root        136 11月  3 21:13 app.tar.gz
-rw-r--r--.  1 root    root        135 11月  3 21:01 hello.txt
drwxr-xr-x.  2 root    root         36 11月  3 21:18 home
drwx------. 15 leopold leopold    4096 11月 11 22:23 leopold
-rw-r--r--.  1 root    root       2123 11月  3 20:23 mycal.txt
-rw-r--r--.  1 root    root    3132991 11月  3 21:16 myhome.tar.gz
-rw-r--r--.  1 root    root    1636090 11月  3 21:08 myhome.zip
-rw-r--r--.  1 root    root          0 11月 14 19:03 test.java
-rw-r--r--.  1 root    root          0 11月 15 17:29 test.txt
[root@localhost home]# 
```





