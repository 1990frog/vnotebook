[TOC]

# docker
## login【登录】
```
>docker login
username:
pwd:
>docker login -u username -p pwd
```
## logout【登出】
```
docker logout [SERVER]
>docker logout dockerhub
Not logged in to dockerhub
```
## info【显示系统信息】
Display system-wide information
```
>docker info
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
## version【版本信息】
```
>docker version

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
## events【实时操作信息（日志输出到当前命令行）】
```
docker events
```
## inspect【底层信息】
Return low-level information on Docker objects
```
>docker inspect [obj]
network
volume
image
container
```
## prune【精简】
```
# 删除 所有未被 tag 标记和未被容器使用的镜像：
>docker image prune
# 删除 所有未被容器使用的镜像：
>docker image prune -a
# 删除 所有停止运行的容器：
>docker container prune
# 删除 所有未被挂载的卷：
>docker volume prune
# 删除 所有网络：
>docker network prune
# 删除 docker 所有资源：
>docker system prune
```
## attach：将本地标准输入，输出和错误流附加到正在运行的容器【不会用】
Attach local standard input, output, and error streams to a running container
```docker
# Docker提供了attach命令来进入Docker容器。
docker attach [container_id]
```
# image
## images【镜像列表】
```docker
>docker images
>docker image ls
```
## search【查询repository镜像】
```
>docker search [image]
```
## pull【拉取镜像】
Pull an image or a repository from a registry
```
>docker search [image]
>docker pull [image]
```
## push【提交镜像】
```
>docker push [image]
```
## rmi【删除镜像】
```
>docker rmi [image]
>docker image rm [image]
```
## history【查看镜像历史构建信息】
```
>docker history [image]
```
## save/load【将一个镜像导出为文件，再使用`load`命令将文件导入为一个镜像，由于会保存该镜像的的所有历史记录，所以比`export`命令导出的文件大】
```
>docker save spring-boot-docker  -o  /home/wzh/docker/spring-boot-docker.tar
>docker load -i spring-boot-docker.tar
```
## export/import【将一个容器导出为文件，再使用`import`命令将容器导入成为一个新的镜像，但是相比`save`命令，容器文件会丢失所有元数据和历史记录，仅保存容器当时的状态，相当于虚拟机快照】
```
>docker import [OPTIONS] file|URL|- [REPOSITORY[:TAG]]
>docker import  my_ubuntu_v3.tar runoob/ubuntu:v4
```
## tag【标记本地镜像，将其归入仓库】
```
>docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
test                latest              beb181dcb46d        6 days ago          1.22MB
>docker tag test:latest test:new
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
test                latest              beb181dcb46d        6 days ago          1.22MB
test                new                 beb181dcb46e        6 days ago          1.22MB
```
## run【运行&拉取】
```
Usage:  docker run [OPTIONS] IMAGE [COMMAND] [ARG...]

Run a command in a new container

Options:
      --add-host list                  Add a custom host-to-IP mapping (host:ip)
  -a, --attach list                    Attach to STDIN, STDOUT or STDERR    #指定标准输入输出内容类型，可选 STDIN/STDOUT/STDERR 三项
      --blkio-weight uint16            Block IO (relative weight), between 10 and 1000, or 0 to disable (default 0)
      --blkio-weight-device list       Block IO weight (relative device weight) (default [])
      --cap-add list                   Add Linux capabilities
      --cap-drop list                  Drop Linux capabilities
      --cgroup-parent string           Optional parent cgroup for the container
      --cidfile string                 Write the container ID to the file
      --cpu-period int                 Limit CPU CFS (Completely Fair Scheduler) period
      --cpu-quota int                  Limit CPU CFS (Completely Fair Scheduler) quota
      --cpu-rt-period int              Limit CPU real-time period in microseconds
      --cpu-rt-runtime int             Limit CPU real-time runtime in microseconds
  -c, --cpu-shares int                 CPU shares (relative weight)
      --cpus decimal                   Number of CPUs
      --cpuset-cpus string             CPUs in which to allow execution (0-3, 0,1)
      --cpuset-mems string             MEMs in which to allow execution (0-3, 0,1)
  -d, --detach                         Run container in background and print container ID    #后台运行容器，并返回容器ID
      --detach-keys string             Override the key sequence for detaching a container
      --device list                    Add a host device to the container
      --device-cgroup-rule list        Add a rule to the cgroup allowed devices list
      --device-read-bps list           Limit read rate (bytes per second) from a device (default [])
      --device-read-iops list          Limit read rate (IO per second) from a device (default [])
      --device-write-bps list          Limit write rate (bytes per second) to a device (default [])
      --device-write-iops list         Limit write rate (IO per second) to a device (default [])
      --disable-content-trust          Skip image verification (default true)
      --dns list                       Set custom DNS servers    #指定容器使用的DNS服务器，默认和宿主一致
      --dns-option list                Set DNS options
      --dns-search list                Set custom DNS search domains
      --domainname string              Container NIS domain name
      --entrypoint string              Overwrite the default ENTRYPOINT of the image
  -e, --env list                       Set environment variables    #设置环境变量
      --env-file list                  Read in a file of environment variables
      --expose list                    Expose a port or a range of ports
      --gpus gpu-request               GPU devices to add to the container ('all' to pass all GPUs)
      --group-add list                 Add additional groups to join
      --health-cmd string              Command to run to check health
      --health-interval duration       Time between running the check (ms|s|m|h) (default 0s)
      --health-retries int             Consecutive failures needed to report unhealthy
      --health-start-period duration   Start period for the container to initialize before starting health-retries countdown (ms|s|m|h)
                                       (default 0s)
      --health-timeout duration        Maximum time to allow one check to run (ms|s|m|h) (default 0s)
      --help                           Print usage
  -h, --hostname string                Container host name    #指定容器的hostname
      --init                           Run an init inside the container that forwards signals and reaps processes
  -i, --interactive                    Keep STDIN open even if not attached    #以交互模式运行容器，通常与 -t 同时使用
      --ip string                      IPv4 address (e.g., 172.30.100.104)
      --ip6 string                     IPv6 address (e.g., 2001:db8::33)
      --ipc string                     IPC mode to use
      --isolation string               Container isolation technology
      --kernel-memory bytes            Kernel memory limit
  -l, --label list                     Set meta data on a container
      --label-file list                Read in a line delimited file of labels
      --link list                      Add link to another container    #添加链接到另一个容器
      --link-local-ip list             Container IPv4/IPv6 link-local addresses
      --log-driver string              Logging driver for the container
      --log-opt list                   Log driver options
      --mac-address string             Container MAC address (e.g., 92:d0:c6:0a:29:33)
  -m, --memory bytes                   Memory limit    #设置容器使用内存最大值
      --memory-reservation bytes       Memory soft limit
      --memory-swap bytes              Swap limit equal to memory plus swap: '-1' to enable unlimited swap
      --memory-swappiness int          Tune container memory swappiness (0 to 100) (default -1)
      --mount mount                    Attach a filesystem mount to the container
      --name string                    Assign a name to the container    #为容器指定一个名称
      --network network                Connect a container to a network    #指定容器的网络连接类型，支持 bridge/host/none/container: 四种类型
      --network-alias list             Add network-scoped alias for the container
      --no-healthcheck                 Disable any container-specified HEALTHCHECK
      --oom-kill-disable               Disable OOM Killer
      --oom-score-adj int              Tune host's OOM preferences (-1000 to 1000)
      --pid string                     PID namespace to use
      --pids-limit int                 Tune container pids limit (set -1 for unlimited)
      --privileged                     Give extended privileges to this container
  -p, --publish list                   Publish a container's port(s) to the host    #指定端口映射，格式为：主机(宿主)端口:容器端口
  -P, --publish-all                    Publish all exposed ports to random ports    #随机端口映射，容器内部端口随机映射到主机的高端口
      --read-only                      Mount the container's root filesystem as read only
      --restart string                 Restart policy to apply when a container exits (default "no")
      --rm                             Automatically remove the container when it exits
      --runtime string                 Runtime to use for this container
      --security-opt list              Security Options
      --shm-size bytes                 Size of /dev/shm
      --sig-proxy                      Proxy received signals to the process (default true)
      --stop-signal string             Signal to stop a container (default "SIGTERM")
      --stop-timeout int               Timeout (in seconds) to stop a container
      --storage-opt list               Storage driver options for the container
      --sysctl map                     Sysctl options (default map[])
      --tmpfs list                     Mount a tmpfs directory
  -t, --tty                            Allocate a pseudo-TTY    #为容器重新分配一个伪输入终端，通常与 -i 同时使用
      --ulimit ulimit                  Ulimit options (default [])
  -u, --user string                    Username or UID (format: <name|uid>[:<group|gid>])
      --userns string                  User namespace to use
      --uts string                     UTS namespace to use
  -v, --volume list                    Bind mount a volume    #绑定一个卷
      --volume-driver string           Optional volume driver for the container
      --volumes-from list              Mount volumes from the specified container(s)
  -w, --workdir string                 Working directory inside the container



>docker run --name mynginx -d nginx:latest
>docker run -P -d nginx:latest
>docker run -p 80:80 -v /data:/data -d nginx:latest
>docker run -p 127.0.0.1:80:8080/tcp ubuntu bash
>docker run -it nginx:latest /bin/bash
```
# container（等同一个进程）
## ps【列表】
```docker
>docker ps -a
```
## commit【通过变化后的container创建一个新的image】
```
Usage:  docker commit [OPTIONS] CONTAINER [REPOSITORY[:TAG]]

Create a new image from a container's changes

Options:
  -a, --author string    Author (e.g., "John Hannibal Smith <hannibal@a-team.com>")
  -c, --change list      Apply Dockerfile instruction to the created image
  -m, --message string   Commit message
  -p, --pause            Pause container during commit (default true)
```
## cp【在container与host之间拷贝文件】
Copy files/folders between a container and the local filesystem
## create【创建一个新的container】
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

---

# network
## connect
Connect a container to a network
## create
Create a network
## disconnect
Disconnect a container from a network
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



# config
## create：Create a config from a file or STDIN
## ls：List configs
## rm：Remove one or more configs




# engine
## activate：激活商用版本
Activate Enterprise Edition
## check：检查可用的引擎更新
Check for available engine updates
## update：更新本地引擎
Update a local engine
