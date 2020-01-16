[TOC]

# 简介
在docker中创建镜像最常用的方式，就是使用dockerfile。dockerfile是一个docker镜像的描述文件，我们可以理解成火箭发射的A、B、C、D…的步骤。Dockerfile其内部包含了一条条的指令，每一条指令构建一层，因此每一条指令的内容，就是描述该层应当如何构建。
```Docker
#基于centos镜像
FROM centos
#维护人的信息
MAINTAINER The CentOS Project <gmail.com>
#安装httpd软件包
RUN yum -y update
RUN yum -y install httpd
#开启80端口
EXPOSE 80
#复制网站首页文件至镜像中web站点下
ADD index.html /var/www/html/index.html
#复制该脚本至镜像中，并修改其权限
ADD run.sh /run.sh
RUN chmod 775 /run.sh
#当启动容器时执行的脚本文件
CMD ["/run.sh"]
```
# 命令
## FROM：指定基础镜像
```Docker
FROM <image> [AS <name>]
FROM <image>[:<tag>] [AS <name>]
FROM <image>[@<digest>] [AS <name>]
FROM scratch    #空白镜像（详见dockerhub）
```
## COPY：复制文本
```Docker
COPY <源路径>... <目标路径>
COPY ["<源路径1>",... "<目标路径>"]
#<源路径> 可以是多个、以及使用通配符，通配符规则满足Go的filepath.Match 规则，如：COPY hom* /mydir/    COPY hom?.txt /mydir/
#<目标路径>使用 COPY 指令，源文件的各种元数据都会保留。比如读、写、执行权限、文件变更时间等。
```
## ADD：高级复制文件（对比COPY强在可以自动解压）
```Docker
ADD ubuntu-xenial-core-cloudimg-amd64-root.tar.gz /
#<源路径> 可以是一个 URL ，下载后的文件权限自动设置为 600 。
```
## ARG：构建参数
与ENV不同的是，容器运行时不会存在这些环境变量。
可以用 docker build --build-arg <参数名>=<值> 来覆盖。
```Docker
#Syntax
ARG <name>[=<default value>]

#Default values
ARG user1=someuser

#Scope
FROM busybox
USER ${user:-some_user}
ARG user
USER $user
$ docker build --build-arg user=what_user .

FROM busybox
ARG SETTINGS
RUN ./run/setup $SETTINGS

FROM busybox
ARG SETTINGS
RUN ./run/other $SETTINGS

ENV arg    #声明变量
${arg:default}    #设置默认值
$arg    #调用变量
```
## ENV：设置环境变量
在其他指令中可以直接引用ENV设置的环境变量。
```Docker
ENV <key> <value>
ENV <key1>=<value1> <key2>=<value2>...
#示例：
ENV VERSION=1.0 DEBUG=on NAME="Happy Feet"
```
这个指令很简单，就是设置环境变量而已，无论是后面的其它指令，如RUN ，还 是运行时的应用，都可以直接使用这里定义的环境变量。
这个例子中演示了如何换行，以及对含有空格的值用双引号括起来的办法，这和Shell下的行为是一致的。
定义了环境变量，那么在后续的指令中，就可以使用这个环境变量。比如在官方node镜像 Dockerfile 中，就有类似这样的代码:
```Docker
ENV VERSION=1.0 DEBUG=on \
    NAME="Happy Feet"
ENV NODE_VERSION 7.2.0
RUN curl -SLO "https://nodejs.org/dist/v$NODE_VERSION/node-v$NOD
E_VERSION-linux-x64.tar.xz" \
  && curl -SLO "https://nodejs.org/dist/v$NODE_VERSION/SHASUMS25
6.txt.asc" \
  && gpg --batch --decrypt --output SHASUMS256.txt SHASUMS256.tx
t.asc \
  && grep " node-v$NODE_VERSION-linux-x64.tar.xz\$" SHASUMS256.t
xt | sha256sum -c - \
  && tar -xJf "node-v$NODE_VERSION-linux-x64.tar.xz" -C /usr/loc
al --strip-components=1 \
  && rm "node-v$NODE_VERSION-linux-x64.tar.xz" SHASUMS256.txt.as
c SHASUMS256.txt \
  && ln -s /usr/local/bin/node /usr/local/bin/nodejs
```
## LABEL：设置metadata（The LABEL instruction adds metadata to an image）
LABEL 指令会添加元数据到镜像。LABEL是以键值对形式出现的。为了在LABEL的值里面可以包含空格，你可以在命令行解析中使用引号和反斜杠。
```Docker
LABEL <key>=<value> <key>=<value> <key>=<value> ...

LABEL "com.example.vendor"="ACME Incorporated"
LABEL com.example.label-with-value="foo"
LABEL version="1.0"
LABEL description="This text illustrates \
that label-values can span multiple lines."
```
## WORKDIR：指定工作目录
```Docker
WORKDIR /a
WORKDIR b
WORKDIR c
RUN pwd    #输出/a/b/c

#The WORKDIR instruction can resolve environment variables previously set using ENV. You can only use environment variables explicitly set in the Dockerfile. For example:
ENV DIRPATH /path
WORKDIR $DIRPATH/$DIRNAME
RUN pwd
```
## VOLUME：定义匿名卷
volume也是绕过container的文件系统，直接将数据写到host机器上，只是volume是被docker管理的，docker下所有的volume都在host机器上的指定目录下/var/lib/docker/volumes。
```Docker
VOLUME ["<路径1>", "<路径2>"...]
VOLUME <路径>
EXPOSE：暴露端口
```
将my-volume挂载到container中的/mydata目录
```
docker run -it -v my-volume:/mydata alpine sh
```
然后可以查看到给my-volume的volume
```Docker
docker volume inspect my-volume
[
    {
        "CreatedAt": "2018-03-28T14:52:49Z",
        "Driver": "local",
        "Labels": null,
        "Mountpoint": "/var/lib/docker/volumes/my-volume/_data",
        "Name": "my-volume",
        "Options": {},
        "Scope": "local"
    }
]
```
可以看到，volume在host机器的目录为/var/lib/docker/volumes/my-volume/_data。此时，如果my-volume不存在，那么docker会自动创建my-volume，然后再挂载。
也可以不指定host上的volume：
```Docker
docker run -it -v /mydata alpine sh
```
此时docker将自动创建一个匿名的volume，并将其挂载到container中的/mydata目录。匿名volume在host机器上的目录路径类似于：
`/var/lib/docker/volumes/300c2264cd0acfe862507eedf156eb61c197720f69e7e9a053c87c2182b2e7d8/_data。`
除了让docker帮我们自动创建volume，我们也可以自行创建：
```Docker
docker volume create my-volume-2
```
然后将这个已有的my-volume-2挂载到container中:
```Docker
docker run -it -v my-volume-2:/mydata alpine sh
```
需要注意的是，与bind mount不同的是，如果volume是空的而container中的目录有内容，那么docker会将container目录中的内容拷贝到volume中，但是如果volume中已经有内容，则会将container中的目录覆盖

在Dockerfile中，我们也可以使用VOLUME指令来申明contaienr中的某个目录需要映射到某个volume：
```Docker
#Dockerfile
VOLUME /foo
```
这表示，在docker运行时，docker会创建一个匿名的volume，并将此volume绑定到container的/foo目录中，如果container的/foo目录下已经有内容，则会将内容拷贝的volume中。也即，Dockerfile中的VOLUME /foo与docker run -v /foo alpine的效果一样。

Dockerfile中的VOLUME使每次运行一个新的container时，都会为其自动创建一个匿名的volume，如果需要在不同container之间共享数据，那么我们依然需要通过docker run -it -v my-volume:/foo的方式将/foo中数据存放于指定的my-volume中。

因此，VOLUME /foo在某些时候会产生歧义，如果不了解的话将导致问题。
## EXPOSE <端口1> [<端口2>...]
```Docker
EXPOSE <port> [<port>/<protocol>...]
EXPOSE 80/tcp
EXPOSE 80/udp
docker run -p 80:80/tcp -p 80:80/udp ...
#EXPOSE 仅仅是声明容器打算使用什么端口而已，并不会自动在宿主进行端口映射。
```
## RUN：执行命令并创建新的Image Layer
RUN命令执行命令并创建新的镜像层，通常用于安装软件包
```Docker
#run可以写多个，每一个指令都会建立一层，所以正确写法如下
RUN buildDeps='gcc libc6-dev make' \
         && apt-get update \
         && apt-get install -y $buildDeps \
         && wget -O redis.tar.gz "http://download.redis.io/releases/redis-3.2.5.tar.gz" \
         && mkdir -p /usr/src/redis \
         && tar -xzf redis.tar.gz -C /usr/src/redis --strip-components=1 \
         && make -C /usr/src/redis \
         && make -C /usr/src/redis install \
         && rm -rf /var/lib/apt/lists/* \
         && rm redis.tar.gz \
         && rm -r /usr/src/redis \
         && apt-get purge -y --auto-remove $buildDeps
```
## CMD or ENTRYPOINT
### 命令格式
+ Shell格式：<instruction> <command>。例如：apt-get install python3
+ Exec格式：<instruction> ["executable", "param1", "param2", ...]。例如： ["apt-get", "install", "python3"]
### 差异
CMD命令设置容器启动后默认执行的命令及其参数，但CMD设置的命令能够被docker run命令后面的命令行参数替换（如果定义了多个CMD，只有最后一个执行）
ENTRYPOINT配置容器启动时的执行命令（不会被忽略，一定会被执行，即使运行 docker run时指定了其他命令）
### CMD：设置容器启动后默认执行的命令和参数
```Docker
#shell 格式： CMD <命令>
#exec 格式： CMD ["可执行文件", "参数1", "参数2"...]
CMD ["nginx", "-g", "daemon off;"]
```
### ENTRYPOINT：设置容器启动时运行的命令
同CMD，指定容器启动程序及参数。
通过--entrypoint 参数在运行时替换。

```Docker
用例一：使用CMD要在运行时重新写命令才能追加运行参数，ENTRYPOINT则可以运行时接受新参数。
示例：
FROM ubuntu:16.04
RUN apt-get update \
&& apt-get install -y curl \
&& rm -rf /var/lib/apt/lists/*
ENTRYPOINT [ "curl", "-s", "http://ip.cn" ]

追加-i参数
$ docker run myip -i
......
当前 IP：61.148.226.66 来自：北京市 联通
```
### 总结
+ 使用 RUN 指令安装应用和软件包，构建镜像。
+ 如果 Docker 镜像的用途是运行应用程序或服务，比如运行一个 MySQL，应该优先使用 Exec 格式的 ENTRYPOINT 指令。CMD 可为 ENTRYPOINT 提供额外的默认参数，同时可利用 docker run 命令行替换默认参数。
+ 如果想为容器设置默认的启动命令，可使用 CMD 指令。用户可在 docker run 命令行中替换此默认命令。
## USER：指定当前用户
```
这个用户必须是事先建立好的，否则无法切换。
USER <用户名>
```
## HEALTHCHECK：健康检查
```Docker
HEALTHCHECK [选项] CMD <命令> ：设置检查容器健康状况的命令
HEALTHCHECK NONE ：如果基础镜像有健康检查指令，使用这行可以屏蔽掉其健康检查指令

HEALTHCHECK 支持下列选项：
    --interval=DURATION (default: 30s)    #两次健康检查的间隔，默认为 30 秒；
    --timeout=DURATION (default: 30s)    #健康检查命令运行超时时间，如果超过这个时间，本次健康检查就被视为失败，默认 30 秒；
    --start-period=DURATION (default: 0s)
    --retries=N (default: 3)    #当连续失败指定次数后，则将容器状态视为 unhealthy ，默认 3次。

#示例
FROM nginx
RUN apt-get update && apt-get install -y curl && rm -rf /var/lib/apt/lists/*
HEALTHCHECK --interval=5m --timeout=3s \
  CMD curl -f http://localhost/ || exit 1
```
## ONBUILD：触发
```Docker
ONBUILD [INSTRUCTION]
```
ONBUILD指令可以为镜像添加触发器。其参数是任意一个Dockerfile 指令。

当我们在一个Dockerfile文件中加上ONBUILD指令，该指令对利用该Dockerfile构建镜像（比如为A镜像）不会产生实质性影响。

但是当我们编写一个新的Dockerfile文件来基于A镜像构建一个镜像（比如为B镜像）时，这时构造A镜像的Dockerfile文件中的ONBUILD指令就生效了，在构建B镜像的过程中，首先会执行ONBUILD指令指定的指令，然后才会执行其它指令。

需要注意的是，如果是再利用B镜像构造新的镜像时，那个ONBUILD指令就无效了，也就是说只能再构建子镜像中执行，对孙子镜像构建无效。其实想想是合理的，因为在构建子镜像中已经执行了，如果孙子镜像构建还要执行，相当于重复执行，这就有问题了。

利用ONBUILD指令,实际上就是相当于创建一个模板镜像，后续可以根据该模板镜像创建特定的子镜像，需要在子镜像构建过程中执行的一些通用操作就可以在模板镜像对应的dockerfile文件中用ONBUILD指令指定。 从而减少dockerfile文件的重复内容编写。
## STOPSIGNAL：停止信号
```Docker
STOPSIGNAL signal
```
## MAINTAINER：设置作者信息
```Docker
MAINTAINER <name>
```
The MAINTAINER instruction sets the Author field of the generated images. The LABEL instruction is a much more flexible version of this and you should use it instead, as it enables setting any metadata you require, and can be viewed easily, for example with docker inspect. To set a label corresponding to the MAINTAINER field you could use:
```Docker
LABEL maintainer="1990frog@gmail.com"
```
This will then be visible from docker inspect with the other labels.

