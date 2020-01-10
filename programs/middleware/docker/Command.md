[TOC]

# docker
## login：登录例如（dockerhub这种）
Log in to a Docker registry
## logout：登出
Log out from a Docker registry
## events：获取docker实时操作信息
Get real time events from the server
```docker
docker events
```
## info：显示系统信息
Display system-wide information
```docker
docker info
Server:
 Containers: 1
  Running: 1
  Paused: 0
  Stopped: 0
 Images: 5
 Server Version: 19.03.5-ce
 Storage Driver: overlay2
  Backing Filesystem: extfs
  Supports d_type: true
  Native Overlay Diff: false
 Logging Driver: json-file
 Cgroup Driver: cgroupfs
 Plugins:
  Volume: local
  Network: bridge host ipvlan macvlan null overlay
  Log: awslogs fluentd gcplogs gelf journald json-file local logentries splunk syslog
 Swarm: inactive
 Runtimes: runc
 Default Runtime: runc
 Init Binary: docker-init
 containerd version: d50db0a42053864a270f648048f9a8b4f24eced3.m
 runc version: d736ef14f0288d6993a1845745d6756cfc9ddd5a
 init version: fec3683
 Security Options:
  seccomp
   Profile: default
 Kernel Version: 5.4.6-2-MANJARO
 Operating System: Manjaro Linux
 OSType: linux
 Architecture: x86_64
 CPUs: 4
 Total Memory: 15.52GiB
 Name: root
 ID: NC3A:DP67:LVHZ:YRM2:ZIPC:Q7FG:LSKH:WDKV:D5HD:G5ZE:WZST:P5W3
 Docker Root Dir: /var/lib/docker
 Debug Mode: false
 Registry: https://index.docker.io/v1/
 Labels:
 Experimental: false
 Insecure Registries:127.0.0.0/8
 Live Restore Enabled: false
```
## inspect：返回有关Docker对象的底层信息
Return low-level information on Docker objects
```docker
docker inspect [container_id/container_name]
```
## attach：将本地标准输入，输出和错误流附加到正在运行的容器
Attach local standard input, output, and error streams to a running container
```docker
# Docker提供了attach命令来进入Docker容器。
docker attach [container_id]
```
## search：DockerHub查询镜像
Search the Docker Hub for images
```docker
docker search [image_name]
```
## version：显示docker版本信息
Show the Docker version information
```docker
docker version

Client:
 Version:           19.03.5-ce
 API version:       1.40
 Go version:        go1.13.4
 Git commit:        633a0ea838
 Built:             Fri Nov 15 03:19:09 2019
 OS/Arch:           linux/amd64
 Experimental:      false

Server:
 Engine:
  Version:          19.03.5-ce
  API version:      1.40 (minimum version 1.12)
  Go version:       go1.13.4
  Git commit:       633a0ea838
  Built:            Fri Nov 15 03:17:51 2019
  OS/Arch:          linux/amd64
  Experimental:     false
 containerd:
  Version:          v1.3.2.m
  GitCommit:        d50db0a42053864a270f648048f9a8b4f24eced3.m
 runc:
  Version:          1.0.0-rc9
  GitCommit:        d736ef14f0288d6993a1845745d6756cfc9ddd5a
 docker-init:
  Version:          0.18.0
  GitCommit:        fec3683
```

# engine
## activate：激活商用版本
Activate Enterprise Edition
## check：检查可用的引擎更新
Check for available engine updates
## update：更新本地引擎
Update a local engine

# builder
## build：通过dockerfile构建一个image
Build an image from a Dockerfile
```config
Usage:  docker build [OPTIONS] PATH | URL | -

Build an image from a Dockerfile

Options:
  -c, --cpu-shares int          CPU shares (relative weight)
  -f, --file string             Name of the Dockerfile (Default is 'PATH/Dockerfile')
  -m, --memory bytes            Memory limit
  -q, --quiet                   Suppress the build output and print image ID on success
  -t, --tag list                Name and optionally a tag in the 'name:tag' format
```
## prune：删除构建缓存
```docker
# 删除 所有未被 tag 标记和未被容器使用的镜像：
docker image prune
# 删除 所有未被容器使用的镜像：
docker image prune -a
# 删除 所有停止运行的容器：
docker container prune
# 删除 所有未被挂载的卷：
docker volume prune
# 删除 所有网络：
docker network prune
# 删除 docker 所有资源：
docker system prune
```
# config
## create：Create a config from a file or STDIN
## inspect：Display detailed information on one or more configs
## ls：List configs
## rm：Remove one or more configs


# image
## import：从压缩文件中创建一个镜像
Import the contents from a tarball to create a filesystem image
## inspect：显示image的详细信息
Display detailed information on one or more images
## save：保存镜像到本地
Save one or more images to a tar archive (streamed to STDOUT by default)
```docker
docker save spring-boot-docker  -o  /home/wzh/docker/spring-boot-docker.tar
```
## load：加载本地镜像
Load an image from a tar archive or STDIN
```docker
docker load -i spring-boot-docker.tar
```
## prune：删除未使用的镜像
Remove unused images
## tag：标记本地镜像，将其归入某一仓库
Create a tag TARGET_IMAGE that refers to SOURCE_IMAGE
Update configuration of one or more containers
## history：查看指定镜像的创建历史
Show the history of an image
```docker
docker history imageIdOrName
```
## images：获取镜像列表（等同于docker image ls）
```docker
docker images
```
## import：从归档文件中创建镜像
Import the contents from a tarball to create a filesystem image
```docker
docker import [OPTIONS] file|URL|- [REPOSITORY[:TAG]]
docker import  my_ubuntu_v3.tar runoob/ubuntu:v4
docker images runoob/ubuntu:v4
```
## pull：从repository（例如dockerhub）中拉取一个image
Pull an image or a repository from a registry
```docker
docker search image
docker pull image
```
## push：提交个image
Push an image or a repository to a registry
## rmi：删除镜像（等同docker image rm）
Remove one or more images
```java
docker rmi image
```
## run：运行一个镜像（集成从仓库拉取）
-d 后台执行
Run a command in a new container
```
Usage:  docker run [OPTIONS] IMAGE [COMMAND] [ARG...]

Run a command in a new container

Options:
  -a, --attach list                    Attach to STDIN, STDOUT or STDERR
  -c, --cpu-shares int                 CPU shares (relative weight)
  -d, --detach                         Run container in background and print
  -e, --env list                       Set environment variables
  -h, --hostname string                Container host name
  -i, --interactive                    Keep STDIN open even if not attached
  -l, --label list                     Set meta data on a container
  -m, --memory bytes                   Memory limit
  -p, --publish list                   Publish a container's port(s) to the host
  -P, --publish-all                    Publish all exposed ports to random ports
  -t, --tty                            Allocate a pseudo-TTY
  -u, --user string                    Username or UID (format:
  -v, --volume list                    Bind mount a volume
  -w, --workdir string                 Working directory inside the container
```



# container
## ps：container列表
List containers
```docker
docker ps -a
```
## commit：通过变化后的container创建一个新的image
Create a new image from a container's changes
```docker
Usage:  docker commit [OPTIONS] CONTAINER [REPOSITORY[:TAG]]

Create a new image from a container's changes

Options:
  -a, --author string    Author (e.g., "John Hannibal Smith <hannibal@a-team.com>")
  -c, --change list      Apply Dockerfile instruction to the created image
  -m, --message string   Commit message
  -p, --pause            Pause container during commit (default true)
```
## cp：在container与host之间拷贝文件
Copy files/folders between a container and the local filesystem
## create：创建一个新的container
Create a new container
## diff：Inspect changes to files or directories on a container's filesystem
Inspect changes to files or directories on a container's filesystem# context
## exec：在一个运行中的container中执行一个命令行
Run a command in a running container
```config
Usage:  docker exec [OPTIONS] CONTAINER COMMAND [ARG...]

Run a command in a running container

Options:
  -d, --detach               Detached mode: run command in the background
      --detach-keys string   Override the key sequence for detaching a container
  -e, --env list             Set environment variables
  -i, --interactive          Keep STDIN open even if not attached
      --privileged           Give extended privileges to the command
  -t, --tty                  Allocate a pseudo-TTY
  -u, --user string          Username or UID (format: <name|uid>[:<group|gid>])
  -w, --workdir string       Working directory inside the container
```
## export：Export a container's filesystem as a tar archive
Export a container's filesystem as a tar archive
## inspect：Display detailed information on one or more containers
## logs：获取container的日志
Fetch the logs of a container
## ls：container列表
## port：获取container端口
List port mappings or a specific mapping for the container
```docker
docker port [container_id/container_name]
1521/tcp -> 0.0.0.0:49161
22/tcp -> 0.0.0.0:49160
```
## prune：删除全部关闭的container
## rename：更改container名字
Rename a container
```docker
docker rename [old_container_name/id] [new_container_name]
```
## restart：重启container
Restart one or more containers
```docker
docker restart [container_name/id]
```
## rm：删除container
Remove one or more containers
```docker
docker rm [container_name/id]
```
## start：启动一个或多个container
Start one or more stopped containers
## stop：停止一个或多个container
Stop one or more running containers
## kill：干掉一个或多个运行中的container
Kill one or more running containers
## stats：实时监控容器资源数据统计
Display a live stream of container(s) resource usage statistics
```docker
docker stats
```
## top：显示container的pid
Display the running processes of a container
```docker
docker top [container_name/id]
```
## pause：暂停container的全部进程
Pause all processes within one or more containers
```docker
docker pause [container_name/id]
```
## unpause：取消暂停container的全部进程
Unpause all processes within one or more containers
```docker
docker unpause [container_name/id]
```
## update：Update configuration of one or more containers
## wait：阻塞运行直到容器停止，然后打印出它的退出代码
Block until one or more containers stop, then print their exit codes
```docekr
docker wait [container_name/id]
```


# network
## connect
Connect a container to a network
## create
Create a network
## disconnect
Disconnect a container from a network
## inspect
Display detailed information on one or more networks
## ls
List networks
## prune
Remove all unused networks
## rm
Remove one or more networks
# node
## demote      Demote one or more nodes from manager in the swarm
## inspect     Display detailed information on one or more nodes
## ls          List nodes in the swarm
## promote     Promote one or more nodes to manager in the swarm
## ps          List tasks running on one or more nodes, defaults to current node
## rm          Remove one or more nodes from the swarm
## update      Update a node
# plugin
## create      Create a plugin from a rootfs and configuration. Plugin data directory must contain config.json and rootfs directory.
## disable     Disable a plugin
## enable      Enable a plugin
## inspect     Display detailed information on one or more plugins
## install     Install a plugin
## ls          List plugins
## push        Push a plugin to a registry
## rm          Remove one or more plugins
## set         Change settings for a plugin
## upgrade     Upgrade an existing plugin
# secret
## create      Create a secret from a file or STDIN as content
## inspect     Display detailed information on one or more secrets
## ls          List secrets
## rm          Remove one or more secrets
# service
## create      Create a new service
## inspect     Display detailed information on one or more services
## logs        Fetch the logs of a service or task
## ls          List services
## ps          List the tasks of one or more services
## rm          Remove one or more services
## rollback    Revert changes to a service's configuration
## scale       Scale one or multiple replicated services
## update      Update a service
# stack
## deploy      Deploy a new stack or update an existing stack
## ls          List stacks
## ps          List the tasks in the stack
## rm          Remove one or more stacks
## services    List the services in the stack
# swarm
## ca          Display and rotate the root CA
## init        Initialize a swarm
## join        Join a swarm as a node and/or manager
## join-token  Manage join tokens
## leave       Leave the swarm
## unlock      Unlock swarm
## unlock-key  Manage the unlock key
## update      Update the swarm
# system
## df          Show docker disk usage
## events      Get real time events from the server
## info        Display system-wide information
## prune       Remove unused data
# trust
```
Management Commands:
  key         Manage keys for signing Docker images
  signer      Manage entities who can sign Docker images

Commands:
  inspect     Return low-level information about keys and signatures
  revoke      Remove trust for an image
  sign        Sign an image
```
# volume
## create      Create a volume
## inspect     Display detailed information on one or more volumes
## ls          List volumes
## prune       Remove all unused local volumes
## rm          Remove one or more volumes
